---
layout: post
title: "Custom Error Handling With Express Resource Auto-Loading"
date: 2012-04-19 14:05
comments: true
categories:
- node.js
- express
- javascript
---

{% pullquote %}
[Express Resource](https://github.com/visionmedia/express-resource) adds RESTful resource routing on top of the [Express](http://expressjs.com/) framework for [Node.js](http://nodejs.org/). It includes a nice feature called "Auto-Loading" that allows you to define a single `load` method that will automatically be used to load your resource object from a URL id parameter and set it in the request object to be accessed by your conventional RESTful methods that need it. Another great feature of Express Resource is what they call "Content-Negotiation". URLs with format extensions like `.xml` or `.json` automatically set an indicator of the format in the request so your actions can respond accordingly.

However, a problem arises when you use the recommended method for handling and reporting errors from within your `load` method: {" You get no control over how the errors are handled, and the default mechanism is ignorant of the requested format."} This makes it tough to return nice JSON errors, for example, from a REST API that uses this technique.
{% endpullquote %}

<!-- MORE -->

## The Problem

The canonical example in the Express Resource documentation describes implementing a `load` method on your resource that calls a callback with an error and/or the loaded object, like so:

```javascript
exports.load = function(req, id, fn) {
  var user  = yourLookupUserFunction(id);
  var error = user ? null : new Error("User Not Found");
  fn(error, user);
};
```

This is great, except there are a few problems:

1. **You can't override what Express Resource does with this error.** The way Express Resource works here is to use a param callback. If you pass an error, its param callback simply passes the error to the next method in the chain via `next(err)`, which ends up invoking the default error handling before any of your methods or `app.error()` handlers are called. This means that a RESTful API call expecting a JSON response won't get a JSON response. If you want your JSON API 404s to return JSON data in the payload, you are out of luck.

2. **There is no response object passed to your auto-load function.** This makes it difficult to bypass Express Resource's intended error handling and issue a response directly on your own.

3. **The content-negotiation hasn't happened yet at this point in the chain.** This means, even if you could respond directly from your `load` function, you wouldn't be able to use the conventional Express Resource mechanism of testing `req.format` to know whether or not to response with JSON, HTML, XML, whatever.

## The Solution

It turns out there are some simple solutions to these problems. Let's tackle them in reverse order.

1. Even though Express Resource has not parsed the route's format yet, and set `req.format` for you, the core routing framework has already done this as params matching. Instead of `req.format`, for this method you need to use `req.params.format`.

2. Even though Express Resource does not pass the `res` object into your `load` function, node.js has conveniently linked the two together for us. In this method, instead of using `res`, you need to use `req.res`. This may be a little dangerous since it relies on the internal structure of the request object, but this linkage seems relatively stable.

3. Don't have your `load` function call the given callback if there is an error. Only call it in the success case. If there is an error, render the response yourself using the tactics described above for #1 and #2.

## Putting it all together

Here is an example of what all that might look like for your `load` function:

```javascript
exports.load = function(req, id, fn) {
  var user = yourLookupUserFunction(id);
  if (user) {
    fn(null, user);
  }
  else {
    switch (req.params.format) {

    case 'json':
      req.res.json({'error' : 'User Not Found'}, 404);
      break;

    case 'html':
    default:
      // Regular 404...or render your own template if you like
      req.res.send(404);
    }
  }
};
```
