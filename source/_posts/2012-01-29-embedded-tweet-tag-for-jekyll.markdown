---
layout: post
title: "Embedded Tweet Tag For Jekyll"
date: 2012-01-29 15:54
comments: true
categories:
- blogging
- twitter
---

A nice way to quote a tweet in your blog post is to use Twitter's [Embedded Tweet](https://dev.twitter.com/docs/embedded-tweets) feature. Not only does it provide a familiar Twitter style, it also provides features like being able to directly retweet, follow, etc., without leaving your page. For example:

{% tweet https://twitter.com/DEVOPS_BORAT/status/159849628819402752 align='center' %}

If you use Octopress or Jekyll, you can easily embed tweets using the [Jekyll Tweet Tag](https://github.com/scottwb/jekyll-tweet-tag). All you need to do is download [tweet_tag.rb](https://raw.github.com/scottwb/jekyll-tweet-tag/master/tweet_tag.rb) and store it in your Jekyll `plugins` directory.

<!-- MORE -->

Then, when you see a tweet you want to quote, click the "Embed this Tweet" link.

{% img https://img.skitch.com/20120129-r8p92x27s6ipncji5bstxeiuqf.png %}

When that pops up the dialog, choose the "Shortcode" tab, select your desired alignment, and copy the resulting code into your blog post.

{% img https://img.skitch.com/20120130-8mw9jdrwcph98eu9igeanhm13c.png %}

***IMPORANT NOTE:*** Since Twitter generates shortcodes for Wordpress, you still have one more step: convert that code to a Liquid-style tag. All that means is replacing the `[...]` with `{{ "{% ... "}}%}`. For example:

```
{{ "{% tweet https://twitter.com/DEVOPS_BORAT/status/159849628819402752 align='center' "}}%}
```

For full documentation, see the project [README](https://github.com/scottwb/jekyll-tweet-tag/blob/master/README.md).

That's all there is to it. If you're using Octopress with your own fork of the repository, you might be interested in my [pull request](https://github.com/imathis/octopress/pull/399) that imports the `jekyll-tweet-tag` plugin.
