---
layout: post
title: "HSTS on Rails"
date: 2013-02-06 09:47
comments: true
categories:
- ruby
- rails
- security
---

[HTTP Strict Transport Security](http://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) (HSTS) is a recent specification aimed at stopping a certain type of man-in-the-middle attack known as _SSL Stripping_. By default, when a user types "example.com" into their browser, the browser prefixes that with "http://". A man-in-the-middle attack can hijack the connection before the server redirect to HTTPS gets back to the browser, spoofing the site and potentially luring the user into providing sensitive data to the attacker.

You can read a nice explanation of this attack, and how HSTS helps to prevent it [here](http://www.imperialviolet.org/2012/07/19/hope9talk.html).

The [wikipedia page on HSTS](http://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) provides some examples on how to enable this in your web server (apache, nginx, etc). However, when running behind an ELB in Amazon Web Services, where you cannot configure this at the reverse proxy, you may wish to do this in your application.

Here is how to achieve that in Ruby on Rails, using a `before_filter` in your base `ApplicationController`:

``` ruby
class ApplicationController < ActionController::Base
  before_filter :strict_transport_security
  def strict_transport_security
    if request.ssl?
      response.headers['Strict-Transport-Security'] = "max-age=31536000; includeSubDomains"
    end
  end
end
```
