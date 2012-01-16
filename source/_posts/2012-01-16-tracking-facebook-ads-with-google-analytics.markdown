---
layout: post
title: "Tracking Facebook Ads With Google Analytics"
date: 2012-01-16 13:37
comments: true
categories: analytics
---
By default, Google Analytics does not do a very good job at categorizing traffic you receive from placing an ad on an external ad network such as Facebook Ads. In the case of Facebook in particular, traffic referred from your ad will show up as "direct" or generically as "facebook" because of they way the redirect the user to your ad through their own analytics back-end.

The intended way to track through this redirection is using "UTM codes": parameters appended to the URL you use in your ad, that lets Google track that user's behavior on your site for the lifecycle of their visit. This is great, because you want to know more than just how many people have clicked on your ad. You want to know what they do once they get to your site (e.g.: if they buy something).

Below is a quick primer on getting setup so that your Facebook Ads can be properly tracked with Google Analytics.

### 1) Add these lines to your existing Google Analytics javascript code:
{% gist 1623245 %}

### 2) Build a URL with [Google Analytics URL Builder](http://support.google.com/googleanalytics/bin/answer.py?hl=en&answer=55578)

These will be the values you can filter by in Google Analytics. For example, you might do this for an ad placed on Facebook:

* **Website URL**: http://www.postonthewall.com/walls/south-beach
* **Campaign Source**: facebook
* **Campaign Medium**: cpc
* **Campaign Term**: spring break _(i.e.: your search terms)_
* **Campaign Content**: _blank - or use for A/B testing two ads at same time_
* **Campaign Name**: spring_break_south_beach

### 3) Place your add with Facebook

Choosing how to position your ad is outside the scope of this post. Take the URL you generated with the Google Analytics URL Builder, and make an ad on Facebook that points to it. You can start with the [Facebook Ads](https://www.facebook.com/advertising/) page.

### 4) Bonus Points: Setup conversion goals in Google Analytics.

This is also largely outside the scope of what I intended to post here, but it's an important step. In short, setting up conversion goals and funnels will help you calculate the ROI for your ads, and will let you figure out where you are losing users that come to your site through these ads.

You can read more about it [here](http://support.google.com/googleanalytics/bin/answer.py?hl=en&answer=55515).

