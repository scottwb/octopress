---
layout: post
title: "Using AddThis With jQuery Mobile"
date: 2012-01-23 14:38
comments: true
categories:
- jquery
- mobile
- addthis
---

Because of the way AddThis works, it can be tricky to use the [AddThis](http://www.addthis.com/) sharing widget within sections of a page that are dynamically vloaded via AJAX. The main [trick they suggest in their forum](http://www.addthis.com/forum/viewtopic.php?f=8&t=13982&st=0&sk=t&sd=a&start=20) is to re-initialize the script after you've dynamically added or modified an AddThis widget. This can be done with a method like this:

``` javascript
if (window.addthis) {
    window.addthis = null;
}
$.getScript("http://s7.addthis.com/js/250/addthis_widget.js#domready=1");
```

Now...when you're using jQuery Mobile, you are very likely loading each page via AJAX instead of a full page request. This is the default behavior. Say you have a series of photos, and each click of a NEXT or PREV button slides in the next/prev photo in a series, as a full page dislay with AddThis buttons. You will need to reload AddThis every time a new page comes in to view.

<!-- MORE -->

You also have to be careful not to reload AddThis for every page being rendered. If you are using jQuery Mobile's page prefetching, you could get quite a few reloads per page this way. You might also miss a necessary reload of AddThis when the user clicks the BACK button, or transitions to an already-cached page.

Fortunately jQuery Mobile gives an easy event to bind to that gives us just the right hook: `pageshow`. This event is triggered when the transition animation has completed for a transition from one page to another. Just the right time to reload AddThis. Put this all together and you get:

``` javascript
function reloadAddThis() {
    if (window.addthis) {
        window.addthis = null;
    }
    $.getScript("http://s7.addthis.com/js/250/addthis_widget.js#domready=1");
}

$(document).bind('pageshow', function() {
    reloadAddThis();
});
```
