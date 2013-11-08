---
layout: post
title: "Easy Papertrail Deployment Using Rake"
date: 2013-11-08 06:37
comments: true
categories:
- tools
- ruby
- rake
- papertrail
---

[Papertrail](http://papertrailapp.com/) is a great centralized logging service you can use for distributed systems that have numerous processes creating numerous log files, across numerous hosts. Having all your logs in one place, live tail-able, searchable, and archived is key to debugging such systems in production.

There are a few ways to set it up, as documented on their quick start page, such as a system-wide installation via [Chef](http://community.opscode.com/cookbooks/papertrail-rsyslog), or configuring your app to use the syslog protocol. They also provide a convenient Ruby gem called [remote_syslog](http://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-text-log-files-in-unix) that can be configured to read a configured set of log files and send them to Papertrail.

I've found for simple Ruby project structures, it can often be easier to deploy Papertrail by installing this gem with Bundler via your project Gemfile, and then creating a simple set of Rake tasks to manage starting and stopping the service. This way it's self-contained within your application repository, gets deployed with the same mechanism you deploy your application code, and can be used on your development and staging systems just as easily, without any Chef cookbooks or other configuration hassle.

<!-- MORE -->

## Rake-based Deployment

I typically build most of my production deployment with modular Rake tasks. This way your Capistrano/OpsWorks/Chef/whatever deployment tools can invoke Rake tasks -- and you can use these same tasks manually on production and development systems alike.

I have a `papertrail.rake` in my [rake-tasks repository on GitHub](https://github.com/scottwb/rake-tasks) that demonstrates how I use this. The contents are shown below, but the rest of the repository demonstrates the other required ingredients, such as a the papertrail config file. With this file in your `tasks` directory, and the `remote_syslog` gem in your `Gemfile`, you now have access to three simple tasks:

``` bash
$ rake -T papertrail
(in /Users/scottwb/src/rake-tasks)
rake papertrail:start   # Start papertrail remote_syslog daemon.
rake papertrail:status  # Show status of papertrail remote_syslog daemon.
rake papertrail:stop    # Stop papertrail remote_syslog daemon.
```

You can now manually start logging to Papertrail with `rake papertrail:start`...or you can hook up the start/stop tasks to your automated deployment tools.

Here are the contents of the main rakefile for this. See the [rake-tasks repository](https://github.com/scottwb/rake-tasks) for example config file, Gemfile, and directory structure.

## The papertrail.rake File

``` ruby
PAPERTRAIL_CONFIG = File.expand_path("../../config/remote_syslog.yml", __FILE__)
PAPERTRAIL_PID    = File.expand_path("../../tmp/pids/remote_syslog.pid", __FILE__)

namespace :papertrail do

  def papertrail_is_running?
    File.exists?(PAPERTRAIL_PID) && system("ps x | grep `cat #{PAPERTRAIL_PID}` 2>&1 > /dev/null")
  end

  desc "Start papertrail remote_syslog daemon."
  task :start => :stop do
    if papertrail_is_running?
      puts "Papertrail is already running."
    else
      sh "remote_syslog -c #{PAPERTRAIL_CONFIG} --pid-file #{PAPERTRAIL_PID}"
    end
  end

  desc "Stop papertrail remote_syslog daemon."
  task :stop do
    if File.exists? PAPERTRAIL_PID
      sh "kill `cat #{PAPERTRAIL_PID}`"
      rm_f PAPERTRAIL_PID
    end
  end

  desc "Show status of papertrail remote_syslog daemon."
  task :status do
    if papertrail_is_running?
      puts "Papertrail remote_syslog is running"
    else
      puts "Papertrail remote_syslog is stopped"
    end
  end
end
```
