---
layout: post
title: "jQuery Mobile and Rails UJS Method Links"
date: 2012-02-17 19:59
comments: true
categories: 
- rails
- jquery
- mobile
---

Rails UJS and jQuery Mobile do not play nice together when it comes to combining Rails UJS's handling of non-GET/POST links with jQuery Mobile's data attributes such as `data-ajax`, `data-direction`, and `data-transition`. This post demonstrates a quick hack you can use to remedy this.

<!-- MORE -->

The `rails.js` file from [rails/jquery-ujs](https://github.com/rails/jquery-ujs) does some very cool stuff to let you emulate HTTP methods other than GET and POST. It does this by looking for a `data-method` attribute on your links. When it finds one, e.g.: `data-method='delete'`, it creates an invisible form to submit a POST with all your link details and a special `_method=delete` parameter that Rails handles in the backend as if it were a `DELETE` method. Using the `link_to` helper, that normally looks like this:

``` erb
<%= link_to "Delete", @post, :confirm => "You sure?", :method => :delete %>
```

jQuery Mobile loads same-domain links and form submissions via AJAX and provide sexy page transitions. You control how these work by adding data attributes to the `<a>` or `<form>` element. One important one, that can affect the correct operation of your page, is the `data-ajax='false'` attribute. That makes disables the AJAX behavior, and loads the next page from your link or form as a new page. That normally looks like this:

``` erb
<%= link_to "View", @post, "data-ajax" => "false" %>
```

One important time you may wish to exercise both tactics at the same time is in providing a delete link. You want to use a link to generate a DELETE request via Rails UJS, and you want to redirect to a new page without jQuery Mobile loading it via AJAX. This is how you would attempt that:

``` erb
<%= link_to("Delete", @post, :confirm => "You sure?",
            :method => :delete, "data-ajax" => "false" %>
```

Sorry. That won't work.

This is because Rails UJS does it's magic by creating a new `<form>` to submit when the link is clicked. It doesn't know or care about the `data-ajax` attribute, and the form it creates does not have that attribute. Then, when the form is submitted, jQuery Mobile handles it using AJAX by default because the form didn't specify otherwise.

We need Rails UJS to copy the `data-ajax` attribute from the link to the form it creates. This is actually a pretty simple fix (hack) to the `handleMethod` funciton in `rails.js`. To just one-off this particular attribute, you can do somethign like this:

{% gist 1857315 %}

In fact, I've [committed this patch](https://github.com/scottwb/jquery-ujs/commit/4d6bc50c4545ac2f492c1e584bef1e154cd61522) in a branch on my fork of rails/jquery-ujs.

With this addition, the above combination of `:method => :delete` and `"data-ajax" => "false"` work beautifully together. You can imagine doing this for other data attributes you care about such as `data-direction` and `data-transition`.

There's currently an [outstanding issue](https://github.com/rails/jquery-ujs/issues/189) to address this, with some discussion on how to go about this generically. If you would like to see this make it into the main distro, head over to that issue and voice your support.
