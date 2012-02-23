---
layout: post
title: "HTML5 Placeholder Polyfill With jQuery Mobile"
date: 2012-02-22 11:29
comments: true
categories:
- html5
- jquery
- mobile
- internet explorer
---

IE9 doesn't support the `placeholder` attribute on `input` elements. Why does this matter for a jQuery Mobile site? If you want to build a single mobile-first page with jQuery Mobile, and use resposive design techiniques to display the same page on desktop browsers, you might appreciate the ability to use the HTML5 `placeholder` in your design, and still need to support at least IE9.

Using a `placeholder` on an input field is a nice way to save space on a mobile form. It lets you forgo the field labels while still telling the user what the fields are for. In many cases its utility goes beyond labeling; you might need it to express input requirements such as what unit of measurement to use in a "distance" field. This is just as useful in a desktop design as it is in a mobile design.

There is a growing movement of creating polyfills for this sort of thing - especially targeted at Internet Explorer. If you're not already using something like [Modernizr](http://www.modernizr.com/) to fulfill all your HTML5 polyfill needs, and just want a solution for the `placeholder` attribute, you're in luck. The Modernizr wiki has a [list of polyfills](https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-browser-Polyfills). You'll see that there are quite a few placeholder polyfills...but they are not all created equal.

<!-- MORE -->

I tried a few of these, and read through the code, and realized that a few of them have some problems. Problems like not supporting password fields, or incorrectly styling non-focused input placeholders. After reviewing these, I decided that the best one to go with was [jquery-placeholder](http://mths.be/placeholder) by [Mathias Bynens](http://mathiasbynens.be/). This one is easy to use, no-ops on browsers that natively support input placeholders, and nicely handles all the corner cases I tested for.

To use jquery-placeholder, simply download `jquery.placeholder.js` from the [project page](http://mths.be/placeholder) and make sure it gets included in your page after jQuery. Then you'll need to apply it to every element you want with a jQuery statement like:

``` javascript
$(selector).placeholder();
```

If you want to use this with jQuery Mobile and its AJAX page-loading mode, you'll need to make sure to execute this statement on every page that gets loaded. You can do that in your main javascript for the document with something like this:

``` javascript
$(document).bind('pageshow', function() {
  var activePage = $('.page.ui-page-active');
  activePage.find('input[placeholder], textarea[placeholder]').placeholder();
});
```

From there, you may notice that the placeholder looks different on all the different browsers. You can style this to be consistent with CSS. You'll need to be a bit repetitive here, because of the way putting the placeholder selectors on the same line breaks some of the browsers. If you use a CSS preprocessor this gets a little easier. Here is an example using Sass's SCSS syntax:

``` scss
input, textarea {
  &.placeholder {
    color: #A9A9A9;
  }
  &:-moz-placeholder {
    color: #A9A9A9;
  }
  &::-webkit-input-placeholder {
    color: #A9A9A9;
  }
}
```
