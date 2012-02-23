---
layout: post
title: "A Better Way To Add Mobile Pages To A Rails Site"
date: 2012-02-23 10:42
comments: true
categories: 
- ruby
- rails
- mobile
---

Having a mobile version of your website is a pretty common thing these days. Doing it with Ruby on Rails seems pretty common as well. Yet there seems to be a lot of misguidance on the web when you search google for advice on making mobile sites in Rails. There are two prevailing suggestions for accomplishing this that I think are undesirable. I've come up with another variation on this that I think is more maintainable and better for the end user.

But first...some background...

_**TL;DR:** Don't use custom MIME formats or domain redirects. Use custom view paths. Skip to "The Final Solution" at the bottom, if you don't care why._

<!-- MORE -->

## Adding a Custom Format Sucks

I don't like everyone's suggestion of adding a `:mobile` or `:iphone` MIME type like this:

```ruby
Mime::Type.register_alias "text/html", :mobile
```

These solutions detect the User-Agent in a `before_filter` and set the request format like this:

```ruby
request.format = :mobile
```

This sucks if you want to use any of your partials for both mobile and desktop pages, because they are considered to be different formats. Say you have a template `show.html.haml` for the desktop version of the page, and a `show.mobile.haml` template for the mobile version. You can't have them both render the same partial. Now imagine you have a common footer you want to use in both version. You'd like to just `render :partial => 'footer'` and have it work. But it doesn't. If your partial is named `_footer.html.haml`, rendering the mobile template will complain that it can't find `_footer.mobile.haml`. You're stuck maintaining two identical copies of this partial. This _really_ sucks if you have a lot of these kinds of partials.

Some people suggest removing the format in the filename, so you'd have `_footer.haml`. I am not fond of this solution.

## Adding a Custom Domain Sucks

Don't you hate it when you see a link to an article on Twitter and click it on your desktop, only to be taken to http://m.whatever.com/ because someone shared this link from their mobile device? Now you're reading a mobile version of this article full screen on your desktop and it looks ridiculous. Or you hit a full version URL from your mobile device and have to suffer yet another redirect. As a user, I would prefer to see one page that looks mobile-friendly on a mobile device, and looks like a full version on a desktop. As a developer, redirecting seems like a cop-out. It also feels like it violates a good RESTful design. There should be one URL for this resource and its view should be tailored to the device on which I am viewing it.

## There Is A Better Way

I like to leverage the same `before_filter` concept of the custom format solution to detect whether or not you are on a mobile device. You can even add a check for a query param that allows a request to set a flag in the session that overrides the mobile-or-not setting. The first step to this is to build some filters and helpers into your `ApplicationController`:

```ruby
def check_for_mobile
  session[:mobile_override] = params[:mobile] if params[:mobile]
end

def mobile_device?
  if session[:mobile_override]
    session[:mobile_override] == "1"
  else
    # Season this regexp to taste. I prefer to treat iPad as non-mobile.
    (request.user_agent =~ /Mobile|webOS) && (request.user_agent !~ /iPad/)
  end
end
helper_method :mobile_device?
```

With these in your `ApplicationControlelr`, you can add `before_filter :check_for_mobile` to any controller/action and have it detect whether or not a request is mobile, or is forced to be mobile (or not) with a query parameter `mobile=1` (or `mobile=0`). You also have a `mobile_device?` method that you can call from any controller or view to see if you are currently rendering a mobile-formatted page. (This is rarely needed, but can be handy in certain situations.)

The next step is to tell Rails to render mobile versions of the templates if the request is deemed to be from a mobile device. Rather than using a custom format, use a ***custom view path***. This is the trick used by Rails engines and plugins to extend the app with its own view templates, while still allowing them to be overridden by the app. To make this work, you need to ***create a separate directory structure for mobile view templates*** and add it to the front of the view load path. This way, you can still use the `:html` format, and can still share templates between mobile and desktop templates. As an added bonus, you can still serve full versions of pages to mobile devices if you haven't implemented the mobile version yet. This is huge if you're trying to incrementally add mobile-friendly pages to an existing desktop-oriented site.

For purposes of this example, I'll call this parallel mobile views directory `views_mobile`. The way you prepend that to the view load path is with the `prepend_view_path` method. I prefer to put this in a method in `ApplicationController` that I can call from the `check_for_mobile` filter if a mobile device is detected, or that I can use directly as it's own filter:

```ruby
def prepare_for_mobile
  prepend_view_path Rails.root + 'app' + 'views_mobile'
end
```

Now, if you use `before_filter :prepare_for_mobile` on any action, it will _always_ be treated as mobile, rendering templates from your `app/views_mobile` directory tree if they exist, falling back to those in your `app/views` directory tree if they don't. This is great if you have a _mobile-first responsive design_ for your page that you want to always serve to mobile and desktop devices, but still have other pages (and layouts) that are fully designed for the desktop that you don't want to mix with.

The last step is to augment the `check_for_mobile` filter method to call `prepare_for_mobile` if it detects a mobile device. That way you can use `before_filter :check_for_mobile` on methods that have two versions. Those actions will render the mobile version from `app/views_mobile` for mobile devices and from `app/views` for non-mobile devices -- and they can both render the same shared partials living in `app/views`.

## The Final Solution

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  def check_for_mobile
    session[:mobile_override] = params[:mobile] if params[:mobile]
    prepare_for_mobile if mobile_device?
  end

  def prepare_for_mobile
    prepend_view_path Rails.root + 'app' + 'views_mobile'
  end

  def mobile_device?
    if session[:mobile_override]
      session[:mobile_override] == "1"
    else
      # Season this regexp to taste. I prefer to treat iPad as non-mobile.
      (request.user_agent =~ /Mobile|webOS) && (request.user_agent !~ /iPad/)
    end
  end
  helper_method :mobile_device?
end

# app/controllers/posts_controller.rb
class PostsController < ApplicationController
  # Render mobile or desktop depending on User-Agent for these actions.
  before_filter :check_for_mobile, :only => [:new, :edit]

  # Always render mobile versions for these, regardless of User-Agent.
  before_filter :prepare_for_mobile, :only => :show
end
```
