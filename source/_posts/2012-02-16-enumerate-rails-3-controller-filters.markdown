---
layout: post
title: "Enumerate Rails 3 Controller Filters"
date: 2012-02-16 22:13
comments: true
categories:
- rails
- ruby
---
There may come a time that you wish to programatically get a list of what before_filters, after_filters, and around_filters are set on a given controller class in Rails. In Rails 2, there was a `filter_chain` method on `ActionController::Base` that you could use to do this. That method is gone in Rails 3 in favor of something much more magical.

One reason for wanting to do this is simply for debugging. Sometimes you just need to sanity check that filters are being applied in the order you think they are. Especially when they're being added dynamically by various plugins.

In Rails 3, I find it convenient to add a few utility methods to `ApplicationController` that list out the symbol names of all the filters on any given controller:

{% gist 1851142 %}

With that in hand, you can do something like this:

``` ruby
ree-1.8.7-2011.03 :002 > ap PostsController.before_filters
[
    [0] :verify_authenticity_token,
    [1] "_callback_before_1",
    [2] :set_geokit_domain,
    [3] :block_internet_explorer,
    [4] :add_authenticity_token,
    [5] :always_mobile,
    [6] :find_post,
    [7] :set_meta_tags
]
```
