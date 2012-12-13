---
layout: post
title: "Provision and Bootstrap Windows EC2 Instances With Chef"
date: 2012-12-13 05:17
comments: true
categories:
- chef
- ruby
- ec2
- windows
---

This post illustrates how you can have a single script on your workstation (yes, of course, it's a Mac) that provisions a new Windows EC2 instance and bootstraps it using Opscode Chef -- written from the point of view of someone who is used to doing this all the time with ease for Linux instances using the `knife-ec2` gem. I'll assume the reader:

  * has a basic working knowledge of Opscode Chef
  * is using Hosted Chef
  * already has a working chef-repo workstation with `knife` configured
  * already has (or can figure out) `knife-ec2` installed and configured with AWS API credentials
  * is on their own for creating actual cookbooks and roles to configure their Windows instances

This is fairly easy to do with Linux instances. Using `knife ec2 server create` and a bunch of parameters, a single command provisions a new Linux instance in EC2, waits for it to come up, connects to it over SSH using the specified key pair, installs `chef-client`, and bootstraps the node using the specified run_list. Done.

However, things are not so simple for Windows Server instances.

<!-- MORE -->

Working with Windows instances in EC2 using Chef presents a few hurdles:

  * Windows doesn't natively support SSH.
  * The `knife ec2 server create` command waits for the instance to accept SSH connections. There is no option to circumvent this.
  * Windows takes forever to provision.
  * Windows instances typically get a random Administrator password generated for them that takes over 15 minutes to retrieve.
  * The `knife-windows` gem provides a `knife bootstrap windows winrm` command that can bootstrap an _existing_ Windows instance with Chef, but cannot provision a _new_ instance.
  * The `knife bootstrap windows winrm` command requires WinRM to be configured on the instance (which it isn't be default), requires the Administrator password of the instnace (which defaults to a random value), and requires the public IP address of the instance (which we don't know until the instance is up).

Below I'll provide a simplified example script that demonstrates how we can hack together a few techniques to create an all-in-one solution for bringing up new Windows Server nodes in the Amazon cloud. Other than `knife` and all the other pre-requisites mentioned above, you'll need to make sure you have the following Ruby gems installed:

``` bash
% gem install knife-ec2
% gem install knife-windows
```

## The Tricks

The Ruby script below uses a few nasty tricks to make this all work:

First, we write a temporary "user data" file to pass to the EC2 API. This gets executed by the new instance when it is first provisioned. There are two tricks we need to stick into the user data file:

  * A BAT `<script>` that configures WinRM (Windows Remote Management), which is what we'll use to connect to and bootstrap the instance.
  * A `<powershell>` script that sets the Administrator password to a value we define. This makes it so we don't have to wait 15+ minutes for EC2 to generate a password for us, and retrieve it manually through the GUI.

Then, we use `knife ec2 server create` to provision the Windows instance to specification, passing in that user data file. This works great for provisioning the instance, but since it was not really designed for Windows and WinRM, there are two tricks we have to employ here:

  * Execute the `knife` command in a sub-process and read its `STDOUT` until we see it output the new instances public IP address. We'll grab that and save that for the next step.
  * This is also our cue to bail out of `knife ec2 server create`. If you were doing this manually, you'd hit `CTRL-C` here, which `knife` is saying "Waiting for sshd" (which is never going to come up). We do that by sending the sub-process a `SIGTERM` signal.

Now, we can't just move on to bootstrapping the node, because it is still booting up, and WinRM may not be configured yet. The trick here is to create a TCP socket to the WinRM port, using the IP address we aquired in the previous step, and wait for it to connect. If it fails to connect, try again until it does. By the time this succeeds, we know WinRM is up and accepting connections. However, we don't know if the rest of the system is ready. Moving on to the next bootstrapping step immediately will run into intermittent errors. I've seen this manifest as an authentication erorr, presumably because we tried to bootstrap over WinRM before the PowerShell script set the password. There may be other mysteries of the Windows universe lurking here as well. My solution: sleep for two minutes. Lame, I know...but so far it is the only thing that has reliably worked.

Finally, we can bootstrap the new running Windows instance with the `knife bootstrap windows winrm` command, using the IP address we acquired, the password we specified in the user data, and the other `knife` params we want to use such as the run_list and environment.

## The Script

Here is a stripped down version of this script demonstrating all these tricks. As you can see, all the custom configuration is hard-coded in constants at the top of the script. You would obviously fill in your own information however you like -- via command-line params, interactive prompts, config files, etc.

Big thanks to my colleauge [Jeremy Groh](https://twitter.com/jgroh9) who paired through this with me and did the bulk of the heavy lifting on the Windows side, especially with the WinRM and password-reset parts.

``` ruby bootstrap-windows.rb https://gist.github.com/4276748 View Gist
#!/usr/bin/env/ruby

require 'socket'

# AWS API Credentials
AWS_ACCESS_KEY_ID     = "your-aws-access-key-id"
AWS_SECRET_ACCESS_KEY = "your-aws-secret-access-key"

# Node details
NODE_NAME         = "webserver-01.example.com"
CHEF_ENVIRONMENT  = "production"
INSTANCE_SIZE     = "m1.large"
EBS_ROOT_VOL_SIZE = 70   # in GB
REGION            = "us-west-2"
AVAILABILITY_ZONE = "us-west-2b"
AMI_NAME          = "ami-46c54c76"
SECURITY_GROUP    = "Web Servers"
RUN_LIST          = "role[base],role[iis]"
USER_DATA_FILE    = "/tmp/userdata.txt"
USERNAME          = "Administrator"
PASSWORD          = "YourAdminPassword"

# Write user data file that sets up WinRM and sets the Administrator password.
File.open(USER_DATA_FILE, "w") do |f|
  f.write <<EOT
<script>
winrm quickconfig -q & winrm set winrm/config/winrs @{MaxMemoryPerShellMB="300"} & winrm set winrm/config @{MaxTimeoutms="1800000"} & winrm set winrm/config/service @{AllowUnencrypted="true"} & winrm set winrm/config/service/auth @{Basic="true"}
</script>
<powershell>
$admin = [adsi]("WinNT://./administrator, user")
$admin.psbase.invoke("SetPassword", "#{PASSWORD}")
</powershell>
EOT
end

# Define the command to provision the instance
provision_cmd = [
  "knife ec2 server create",
  "--aws-access-key-id #{AWS_ACCESS_KEY_ID}",
  "--aws-secret-access-key #{AWS_SECRET_ACCESS_KEY}",
  "--tags 'Name=#{NODE_NAME}'",
  "--environment '#{CHEF_ENVIRONMENT}'",
  "--flavor #{INSTANCE_SIZE}",
  "--ebs-size #{EBS_ROOT_VOL_SIZE}",
  "--region #{REGION}",
  "--availability_zone #{AVAILABILITY_ZONE}",
  "--image #{AMI_NAME}",
  "--groups '#{SECURITY_GROUP}'",
  "--user-data #{USER_DATA_FILE}",
  "--verbose"
].join(" ")

# Run `knife ec2 server create` to provision the new instance and
# read the output until we know it's public IP address. At that point,
# knife is going to wait until the instance responds on the SSH port. Of
# course, being Windows, this will never happen, so we need to go ahead and
# kill knife and then proceed with the rest of this script to wait until
# WinRM is up and we can bootstrap the node with Chef over WinRM.
ip_addr = nil
IO.popen(provision_cmd) do |pipe|
  begin
    while line = pipe.readline
      puts line
      if line =~ /^Public IP Address: (.*)$/
        ip_addr = $1.strip
        Process.kill("TERM", pipe.pid)
        break
      end
    end
  rescue EOFError
    # done
  end
end
if id_addr.nil?
  puts "ERROR: Unable to get new instance's IP address"
  exit -1
end

# Now the new instance is provisioned, but we have no idea when it will
# be ready to go. The first thing we'll do is wait until the WinRM port
# responds to connections.
puts "Waiting for WinRM..."
start_time = Time.now
begin
  s = TCPSocket.new ip_addr, 5985
rescue Errno:ETIMEOUT => e
  puts "Still waiting..."
  retry
end
s.close

# You'd think we'd be good to go now...but NOPE! There is still more Windows
# bootstrap crap going on, and we have no idea what we need to wait on. So,
# in a last-ditch effort to make this all work, we've seen that 120 seconds
# ought to be enough...
wait_time = 120
while wait_time > 0
  puts "Better wait #{wait_time} more seconds..."
  sleep 1
  wait_time -= 1
end
puts "Finally ready to try bootstrapping instance..."

# Define the command to bootstrap the already-provisioned instance with Chef
bootstrap_cmd = [
  "knife bootstrap windows winrm #{ip_addr}",
  "-x #{USERNAME}",
  "-P '#{PASSWORD}'",
  "--environment #{CHEF_ENVIRONMENT}",
  "--node-name #{NODE_NAME}",
  "--run-list #{RUN_LIST}",
  "--verbose"
].join(' ')

# Now we can bootstrap the instance with Chef and the configured run list.
status = system(bootstrap_cmd) ? 0 : -1
exit status
```
