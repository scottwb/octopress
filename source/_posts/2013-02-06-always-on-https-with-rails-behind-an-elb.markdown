---
layout: post
title: "Always-On HTTPS With Rails Behind an ELB"
date: 2013-02-06 11:43
comments: true
categories:
- ruby
- rails
- aws
- security
---

So you want to run your Rails site such that it always uses HTTPS, and you want all HTTP URLs to redirect to their HTTPS counterparts? Typically you use `config.force_ssl = true` in your initializer, or you use `force_ssl` in your controllers. For various reasons having to do with late-binding configuration, I have typically not been able to use the `config.force_ssl` method. This means the easiest way to force the whole site to use HTTPS has been to use `force_ssl` on the base `ApplicationController`, like this:

``` ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  force_ssl
end
```

However...when you deploy this to Amazon EC2 behind an ELB (Elastic Load Balancer), you can run into problems.

<!-- MORE -->

## Problem: ELB Health Check vs Rails force_ssl

Even if you have your ELB configured with your SSL certificate and you have it proxying port 443 to port 80 on your Rails app, you may still have trouble getting the ELB to accept your instance as an upstream server if it cannot get an `HTTP 200 OK` from the health check action.

Once you have your Rails app using a global `force_ssl`, the ELB HealthCheck will hit your server over HTTP (because you don't actually have your Rails server setup as an SSL endpoint), and your server will return it a 301 redirect. This causes the ELB to think your instance is unhealthy and won't proxy any requests to it.

## Solution: Custom HTTP-able Health Check Action

I've found the easiest way to deal with this is to create a special action that you use for the health check, and override the `force_ssl` for that action. Unfortunately, the stock implementation of `ActionController::Base.force_ssl`, when applied globally in the `ApplicationController`, does not allow other controllers to override that setting. That means we have to tackle this in two steps.

First, re-implement the `force_ssl` method to allow controllers to override it:

``` ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  def self.force_ssl(options = {})
    host = options.delete(:host)
    before_filter(options) do
      if !request.ssl? && !Rails.env.development? && !(respond_to?(:allow_http?) && allow_http?)
        redirect_options = {:protocol => 'https://', :status => :moved_permanently}
        redirect_options.merge!(:host => host) if host
        redirect_options.merge!(:params => request.query_parameters)
        redirect_to redirect_options
      end
    end
  end

  force_ssl
end
```

The above is a direct copy of this method from Rails 3.2, with the additional clause: `&& !(respond_to?(:allow_http?) && allow_http?)`. That clause allows any controller to implement an `allow_http?` method, which is executed in the context of a request's `before_filter`. If this method exists and returns `true` for a given request, then it will be allowed to continue over HTTP without being redirected to HTTPS.

For the second part, we need to create an unprotected action that can be used for the health check. The easiest way to do this is with a new controller (and matching route, if necessary):

``` ruby
# app/controllers/heath_check_controller.rb
class HealthCheckController < ApplicationController
  def index
    render :text => "I am alive!\n"
  end

  protected
  def allow_http?
    true
  end
end
```

``` ruby
# config/routes.rb
MyApp::Application.routes.draw do
  get "health_check" => "health_check#index"
  # ...
end
```

Now, all you need to do is change your ELB Health Check to use `/health_check` instead of `/index.html`. This way the ELB will check that your Rails app is responding using HTTP (since that is the appropriate protocol between the ELB and Rails if you are using the ELB as your SSL endpoint). Your instance will register as healthy as long as your Rails app is up, and Rails will redirect all other HTTP traffic to HTTPS.

***UPDATED Oct. 28, 2013:** If you run your own reverse proxy in front of Rails, you can do this in the reverse proxy without having to modify your Rails app. See my [post on doing this with nginx](http://scottwb.com/blog/2013/10/28/always-on-https-with-nginx-behind-an-elb/).*
