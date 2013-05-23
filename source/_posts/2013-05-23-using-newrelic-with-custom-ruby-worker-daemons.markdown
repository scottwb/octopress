---
layout: post
title: "Using NewRelic With Custom Ruby Worker Daemons"
date: 2013-05-23 07:48
comments: true
categories:
- ruby
- tools
- newrelic
---

The NewRelic Ruby Agent comes with great support for Rails, Sinatra, and other frameworks and web servers out of the box. It also supports background jobs for frameworks like DelayedJob and Resque.

But what if you have your own custom background worker mechanism?

It's fairly simple to get NewRelic working to report your custom background workers, but finding the right combination of setup calls in their docs can be a little tricky. The biggest issue is dealing with background tasks that daemonize and fork child worker processes. This is because the NewRelic agent needs to do unique instrumenting, monitoring, and reporting per process. Setting it up that way can be tricky if you're using Bundler or another mechanism to load the `newrelic_rpm` gem *before* the child processes are forked.

<!-- MORE -->

Assuming you are already familiar with the mechanics of Ruby-based daemon processes, here are the key ingredients you need to integrate the NewRelic Ruby Agent:

1. Store your `newrelic.yml` config file somewhere and make a place for its log file to be written.
2. Setup the environment variables `RUBY_ENV`, `NRCONFIG`, and `NEW_RELIC_LOG` to take the place of `RAILS_ENV` and default config and log paths you may be used to in Rails.
3. Require the `newrelic_rpm` gem or add it to your `Gemfile` and require it via Bundler.
4. Add instrumentation to your main job class with `include ::NewRelic::Agent::Instrumentation::ControllerInstrumentation`
5. Add a tracer to your main job execution method, e.g.: `add_transation_tracer :execute, :category => :task`, in your main job class.
6. Before you daemonize and fork child processes, make sure to call `::NewRelic::Agent.manual_start`.
7. In the child process, right after it's been forked, make sure to call `::NewRelic::Agent.after_fork(:force_reconnection => true)`.

This will now make sure that the NewRelic Agent is started correctly for each child process and will report metrics on the `execute` method of your job class.

## Example

While it's not my intention to go into detail on how to build out a daemonized forking worker mechanism, below is a very simple worker script that demonstrates all of these pieces together. It assumes the use of Bundler and a directory structure like this:

```
project_dir
  |
  +--Gemfile
  |
  +--worker.rb
  |
  +--config
  |    |
  |    +--newrelic.yml
  |
  +--log
       |
       +--newrelic_agent.log
```

This example `worker.rb` script forks 4 worker daemon processes, each of which will report timing metrics to NewRelic for the jobs it runs. Note the comments correlating to the bullet points above.

``` ruby
#!/usr/bin/env ruby

# STEP 2:
#
# Setup NewRelic environment before NewRelic gets loaded by Bundler. This
# is necessary because we don't have the luxury of relying on the NewRelic
# defaults that are geared towards Rails.
ENV['RUBY_ENV'] = 'production'
ENV['NRCONFIG'] ||= File.expand_path('../config/newrelic.yml', __FILE__)
ENV['NEW_RELIC_LOG'] ||= File.expand_path('../log/newrelic_agent.log', __FILE__)

# STEP 3:
#
# Setup Bundler and use it to require all the gems from Gemfile, including
# the `newrelic_rpm` gem.
require 'rubygems'
ENV['BUNDLE_GEMFILE'] ||= File.expand_path('../../Gemfile', __FILE__)
require 'bundler/setup' if File.exists?(ENV['BUNDLE_GEMFILE'])
if defined?(Bundler)
  Bundler.require(:default, 'production')
end

class Job
  # STEP 4: Add instrumation to main job class.
  include ::NewRelic::Agent::Instrumentation::ControllerInstrumentation

  def execute
    # do work
  end

  # STEP 5: Add a tracer to the main job execution method
  add_transaction_tracer :execute, :category => :task
end

class Worker
  def run(num_processes = 1)
    # STEP 6: Set NewRelic Agent to manual start before daemonizing.
    ::NewRelic::Agent.manual_start

    # Double fork daemonize so that forked child processes do not get SIGHUP
    # when the controlling tty dies.
    fork and exit
    Process.setsid
    fork and exit

    num_processes.times do
      if pid = Process.fork
        Process.detach(pid)
      else
        # STEP 7: Force NewRelic Agent to reconnect in forked child process.
        ::NewRelic::Agent.after_fork(:force_reconnection => true)

        loop { next_job.execute }
      end
    end
  end

  private

  def next_job
    # TODO: Do whatever you do to get your next Job instance to execute.
    #       Read from a queue, etc...
  end
end

# Run 4 forked worker daemon processes.
Worker.new.run(4)
```
