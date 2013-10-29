---
layout: post
title: "Always-On HTTPS With Nginx Behind an ELB"
date: 2013-10-28 18:42
comments: true
categories: aws, nginx, security
---

A while back, I wrote about [configuring a Rails app to always enforce HTTPS behind an ELB](http://scottwb.com/blog/2013/02/06/always-on-https-with-rails-behind-an-elb/). The main problem is that it's easy to setup the blanket requirement for HTTPS, but when you are behind an ELB, where the ELB is acting as the HTTPS endpoint and only sending HTTP traffic to your server, you break the ability to respond with an `HTTP 200 OK` response for the health check that the ELB needs. This is because your blanket HTTPS enforcement will redirect the ELB's health check from HTTP to HTTPS -- and that redirection is not considered to be a healthy response by the ELB.

The same applies to any server you're running behind an ELB in this fashion.

This posts discusses how to handle the same issue with Nginx.

<!-- MORE -->

In this scenario, we have an ELB accepting HTTPS traffic and proxying it over HTTP in the clear to an Nginx server listening on port 80. We want Nginx to force all requests that were not originally made with HTTPS to redirect to the same URL on HTTPS, _except_ requests for the health check, which the ELB will make directly over HTTP. For this example, we are using Nginx as a reverse proxy to upstream server processes on the same instance, such as a unicorn webserver hosting a Sinatra app. (This would work well for Rails, too).

## The Solution

There are two main components that make up this solution:

1. A specific `location` directive for the health check URL that does not do any HTTPS enforcement.
2. A redirect if the `X-Forwarded-Proto: https` header does not exist.

For best-practice, we can add [HTTP Strict Transport Security](http://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) with the `add_header` directive here too. Below is an example of a simplified nginx config file demonstrating these.

```
upstream unicorn {
  server localhost:3000;
}

server {
  listen 90;
  server_name example.com;
  root /var/www/html;

  # 1) Special, somewhat redundant location to always proxy
  #    the health check to the upstream server, without checking
  #    if the request came in over HTTP or HTTPS.
  location /health_check {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_next_upstream error;
    proxy_pass http://unicorn;
    break;
  }

  # Our main location to proxy everything else to the upstream
  # server, but with the added logic for enforcing HTTPS.
  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_next_upstream error;

    # 2) Any request that did not originally come in to the ELB
    #    over HTTPS gets redirected.
    if ($http_x_forwarded_proto != "https") {
      rewrite ^(.*)$ https://$server_name$1 permanent;
    }

    proxy_pass http://unicorn;

    # Add HTTP Strict Transport Security for good measure.
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains;";
  }
}
```
