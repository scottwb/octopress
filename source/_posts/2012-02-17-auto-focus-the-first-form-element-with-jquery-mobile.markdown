---
layout: post
title: "Auto-Focus The First Form Element With jQuery Mobile"
date: 2012-02-17 10:08
comments: true
categories:
- jquery
- mobile
---

Imagine a series of pages with simple forms on each of them. With jQuery Mobile, you'll usually end up transitioning between these pages using AJAX and nice slide transitions. Since all your forms will end up in the same document, you need to be careful to give them unique IDs if you want to access them from javascript.

One thing I like to do with forms, especially on mobile platforms, is to automatically highlight the first field in the form when it comes in to view. The trick with jQuery Mobile is that with AJAX loading, pre-loading, and BACK/FORWARD buttons, you need to figure out which form is currently in view, and when transitions between forms happen.

Fortunately jQuery Mobile provides us the two facilities we need:

  * The `ui-page-active` CSS class gets applied to the `.page` element that is currently in view.
  * The `pageshow` event gets triggered on the document when a page transition has completed to change which page is in view.

Combine these two, and you can do something like this in your main javascript once for the whole document:

``` javascript
$(document).bind('pageshow', function() {
    $($('.page.ui-page-active form :input:visible')[0]).focus();
});
```

Now, after every page transition, if there is a form on the newly-visible page, the first visible input element automatically receives focus. This works across AJAX, non-AJAX, and BACK/FORWARD page transitions.
