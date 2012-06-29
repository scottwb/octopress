---
layout: post
title: "jQuery Mobile Breaks Your HREFs on iOS Mobile Safari"
date: 2012-06-29 00:00
comments: true
categories:
- jquery
- mobile
- javascript
- rails
- ios
---

Starting with verison 1.1.0RC1, jQuery Mobile changes the `href` of any link to be `#` when it is clicked, but _only on iOS Mobile Safari_. If you try to make a click handler that reads the `href` attribute, you will be surprised to find that it does not get the href you intended. For example:

```javascript
$('a').click(function() {
  console.log($(this).attr('href'));
});
```

When using jQuery Mobile, this code will print out the href you expect on every browser except Mobile Safari on iOS (both real devices and simulators, but _not with other browsers faking their User-Agent_). On Mobile Safari, this will print out that the href is "#".

## The Problem

Obviously this is a problem if you have code that operates in a click handler that reads the clicked element's `href` attribute. jQuery Mobile changes that href before you get a chance to read it.

One common place this is used is with the `link_to` helper in Ruby on Rails, for links that need to use other HTTP methods. For example, the standard RESTful way to delete a resource will be with a link that sets `:method` to `:delete`. Rails handles this with a library called jQuery UJS that needs to read the href from the link when it is clicked.

Unfortunately for Rails users, this means that Rails method links don't work with jQuery Mobile on iOS Mobile Safari.

<!-- MORE -->


## The Reason

The jQuery Mobile team is doing this on purpose, to make it so the address bar on iPhone does not drop down when changing pages (and we all love that feature!). In [this discussion](https://github.com/jquery/jquery-mobile/issues/3777) and [this discussion](https://github.com/jquery/jquery-mobile/issues/3686), they seem to understand that this makes custom click handlers get the wrong href, but decide to go ahead with it anyway.

You can see it happen in this excerpt from the jQuery Mobile 1.1.0 source (reformatted for blog-friendlyness):

```javascript
// By caching the href value to data and switching the href to a #,
// we can avoid address bar showing in iOS. The click handler resets
// the href during its initial steps if this data is present
$( link )
  .jqmData( "href", $( link  ).attr( "href" )  )
  .attr( "href", "#" );
```

## The Solution

Here's the part that seems to be undocumented other than in that comment: _they store the href in a data attribute!_ This makes this problem easy to workaround. Simply change your click handler to check for the data attribute first and fall back to the href:

```javascript
$('a').click(function() {
  console.log($(this).data('href') || $(this).attr('href'));
});
```

That would be easy enough to encapsulate in a nice helper function...

## Applying This To Rails jQuery UJS

If you use the Rails `link_to` helper with the `:method` option set to anything other than `:get`, links are handled by the `jquery_ujs` library that comes with Rails (often referred to as Rails UJS, jQuery UJS, rails.js). You'll notice, for example that none of your `:delete` links work with jQuery Mobile when you are on iOS Mobile Safari. You might see this as a missing route for your `show` or `edit` method depending on where you are linking from.

It turns out the fix is easy. The jquery_ujs code already has a method to encapsulate getting the href from an element, complete with a comment telling you how you can override it. All it does out of the box is read `element.attr('href')`. All we have to do is override it and make it try the data attribute first.

Somewhere after `jquery_ujs.js` is loaded, define this:

```javascript
$.rails.href = function(element) {
  return element.data('href') || element.attr('href');
}
```

Now your Rails `link_to` calls with custom `:method` options such as `:delete` will work just fine with jQuery Mobile 1.1.0 on iOS Mobile Safari, and you don't have to lose that clever bit that keeps the address bar hidden.
