---
layout: post
title: "5 Things Your Code Must Be"
date: 2013-11-10 06:47
comments: true
categories:
- development
- architecture
---

If you work for me, with me, or near me, there are certain qualities of your code that must be met before I will let you get away with calling it "done". Developers love to brag about a piece of code being done the second it sort of works for the main "happy path" use case it was intended for...on their development system. Developers are also optimists when it comes to estimation. Some of that has to do with their enthusiasm for achieving the "done" state based on their own notion of just having solved the base problem they set out to solve.

This creates all kinds of schedule problems, expectation/reality mismatch problems with management, marketing, sales, etc.

There a tons of details that go into the definition of "done", and they vary depending on the project and the organization's software development lifecycle. Rather than try to rigorously enumerate these -- which would be the topic of an entire book -- I tend to lean on 5 keystone requirements that we can assess about the code before we can call it "done". There are many tactical rules for developing good software, but at a strategic level, following these keystone requirements generally leads us to cover most of those details.

So...before you can call it "done", your code module must be:

<!-- MORE -->

### #1 - Testable
This is fairly obvious if you follow TDD/BDD methodologies. But this is also important when it comes to things like daemon services that produce/consume messages via queues. We need to keep an eye on making those services fully self-contained such that we can mock the queue system, as well as the collaborator queue producers/consumers, and still fully integration test the service under test. Enforcing this often eliminates other classes of problems such as code coupling. _If your code is not testable (and covered with tests!), it is not done._


### #2 - Configurable
Maybe small features don't need this day one, but we often find ourselves with things like constants defined very early on. If the overall system has a configuration framework of some sort, we should be using that from day one, and we should be especially aware of config params that vary between test/dev/stag/prod environments, and have these extracted to a config file. This makes it so that your automated deployment tools can manage environment configuration as well. _If your code module cannot be externally configured for different environments as necessary, it is not done._


### #3 - Deployable
Deployability often piggybacks on existing deployment mechanisms for small features, but in a larger sense, I mean "your new image upload processing service isn't done until it has the worker process wrapper, config file, rake/capistrano tasks as necessary, chef cookbook/recipe/role as necessary, etc. that are necessary to deploy and run it in development, staging, and production". You don't get away with throwing code over the fence to the operations team these days. One of the core tenets of the "DevOps" culture is that developers are responsible for the operations aspects of their code. _If you code module cannot be deployed by the automated continuous deployment system, it is not done._


### #4 - Measurable
For some features this just may be logging. Even just at that, it's important this integrates day one with your centralized logging infrastructure (rsyslog, [Papertrail](http://papertrailapp.com/), [Loggly](http://www.loggly.com/), etc). I believe that every unit should emit timing and workload information to a stats collector like `statsd` or [Cube](http://square.github.io/cube/). In a service-oriented architecture, for example, at the very least, each queue and service should be recording message queue wait times, throughput, busy/idle percentage, processing times, etc. Throw that over to something like [nagios](http://www.nagios.org/), [zabbix](http://www.zabbix.com/), [Scout](https://www.scoutapp.com/), or the new custom monitoring charts at [NewRelic](http://newrelic.com/), and you should be able to answer those hard ops questions about where something went wrong in your distributed system much more quickly. _If we cannot measure the operating parameters of your code, it is not done._


### #5 - Monitorable
Maybe this isn't that much different than #4, or maybe this should really be called "alertable". For the most part, when we release a piece of code, we want to know if it is working, and when it stops working. At the simplest level, this is aimed at letting us sleep at night - ideally this includes "restartable". We don't need to manually check if the async mailer workers are down if something like [god](http://godrb.com/) or [monit](http://mmonit.com/monit/) is watching our processes, and when they die, alerting us and restarting them. (This applies at the cloud instance level too.) Being monitorable doesn't just have to apply at the process level though. Going hand-in-hand with measurability, this comes down to defining early the operating bands for our measurements from #4, and when to alert that we've gone outside of them. For example, your "sign up form" feature could be measured and monitored for signups-per-hour. If you have a steady enough visitor rate, there is some low (perhaps zero) that you don't think you should ever reach if your sign up form is functioning properly. Alerting on this proxy variable can serve as a canary in the coal mine, letting you know that an underlying problem is developing. _If we cannot tell when your code is in trouble, and ideally be able to kill and restart it, then it is not done._
