---
layout: post
title: "Show jQuery Mobile Page Loading With Text And Spinner"
date: 2012-06-30 21:47
comments: true
categories: 
- jquery
- mobile
- javascript
---

I like jQuery Mobile's page loading spinner. When you transition between pages it displays a nice subtle spinner while the page loads, only if the page load takes more than a configured number of milliseconds.

So, I wanted to use their page-loading spinner for in-page wait times in my app. The user clicks something, I do some AJAX stuff and show them a nice spinner with a message telling them what's going on until the results come back. jQuery Mobile has javascript functions for this called `$.mobile.showPageLoadingMsg` and `$.hidePageLoadingMsg`. The problem is, you can either show a spinner only, or a text-message only. You can't show both a spinner _and_ a text message, even though they clearly support it. Well, you _can_ do that if you have the global setting `$.mobile.loadingMessageTextVisible = true`, but then you get a text with your normal page-load spinners.

<!-- MORE -->

I've created a workaround for this and included it in my [jquery.mobile.utils](https://github.com/scottwb/jquery.mobile.utils) library. Using this you can call:

```javascript
// Params: jqm theme swatch, and message text
$.mobile.utils.showWaitBox("a", "Hang on while I do work...");
// ... some time later...
$.mobile.utils.hideWaitBox();
```

That will let you leave the standard textless page-loading spinenrs alone, but be able to show a nice themed wait box with a spinner while your app is doing something the user needs to wait for, that looks like this:


{% img center http://img.skitch.com/20120701-met428ntqsxrest228pxgrrxjs.png %}

To use these methods, download the JavaScript or CoffeeScript version of the jquery.mobile.utils library from the [project's github page](https://github.com/scottwb/jquery.mobile.utils) and include it in your jQuery Mobile page after `jquery.mobile-1.1.0.js` is loaded.
