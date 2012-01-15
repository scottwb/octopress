---
layout: post
title: "Twitter Image Search and Download"
date: 2012-01-14 14:03
comments: true
categories:
- twitter
- ruby
- scripts
---
Last night the sunset in the Seattle area was in rare form. You might have thought there was a [double rainbow](http://www.funnyordie.com/videos/dcf83410c7/insane-double-rainbow-guy) by the way everyone was going on about it. My wife even noted that her Facebook feed was filled with pictures of it.

So I thought it would be funny to scour Twitter for all the pictures of it that I could find and post them to Facebook just to be a smart-ass.

I sat down to whip out a quick-and-dirty script to do this for me. By the time it was ready, I had lost the smart-ass urge to post them to Facebook, but what I ended up with turns out to be a handy little tool.

You can grab it from my github page at [https://github.com/scottwb/twitter-image-search](https://github.com/scottwb/twitter-image-search)

It's pretty easy to use. You give it a search term and it creates a directory and downloads a bunch of matching images.

The sunset may have passed, but this morning we have over a millimeter of snow in the Seattle area and all hell is breaking loose. It's SNOWPOCALPSE 2012!!! So, of course I ran this script:

``` bash
$ ./twitter-image-search.rb "seattle #snowpocalypse2012"
```

And you can see from the results, people are clearly a bit worked up about the snow:

{% img https://img.skitch.com/20120114-kbehj5uxyghxr4p9nic6rh7wp3.png %}
