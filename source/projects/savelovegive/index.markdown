---
layout: project_page
title: SaveLoveGive&trade;
date: 2013-11-10 13:43
date_range: 2012 - 2013
favicon: /projects/savelovegive/favicon.png
---

{% img center /projects/savelovegive/hero.png 'SaveLoveGive' 'SaveLoveGive' %}

SaveLoveGive&trade; was the branding campaign for the direct-to-consumer efforts of the VERA&trade; Platform at [Validas](http://www.validas.com). It was a modern, mobile-first response-design web-app that helped users save money on their cell phone bills, with an option to donate a portion of their savings to a charitable cause. We built it from scratch in under six months, on top of our wireless billing analysis engine, VERA&trade;, and launched to an initial media fanfare that lead to over $4 million in consumer savings within the first six weeks of going live, and went on to raise over $100,000 in charitable donations in its first few months.

## My Contribution

As the newly-hired VP of Engineering, I was charged with building and leading the small team that took the existing enterprise backend technology and developed it into a consumer-facing product. This required building a new service API abstraction between the legacy enterprise systems, and designing and developing a new consumer-friendly user interface. Working with [Jeremy Groh](http://linkedin.com/in/jgroh9) leading the backend team, and [Leif Jensen](http://www.linkedin.com/in/leifjensen) creating the market-driven product and feature requirements, my team built the entire front-end system from scratch, meeting ever-changing marketing requirements, driving backend engine changes, and interfacing with third-party charities and payment systems.

In addition to recruiting, hiring, and setting the tone for a new era of tools and technologies at Validas, I was involved in both leading the software engineering team and serving as an individual contributor to the implementation of this project. This was roughly a 6-month project from conception to our public launch featured on [World News with Diane Sawyer](http://abcnews.go.com/WNT/video/real-money-investigates-overspending-cellphone-bills-18215358), Nightline, and a number of other major news outlets -- which made for an interesting overnight scalability challenge!

My biggest areas of individual contribution in software development included:

  * Improved existing enterprise backend architectures to be more modular and scalable in anticipation of our big launch.
  * Architecture that allowed us to scale from 10 hits-per-hour to tens of thousands of hits per minute, in under a minute when the first news story hit the airwaves.
  * Built custom measurement and analysis tools to help steer the product roadmap.
  * Full stack implementation of the web app - Rails, HTML5, CSS, Javascript, etc.
  * Custom continuous deployment and high-availability mechanisms.
  * Technical liaison to charity partners, designing architecture to allow them to be easily plugged in to our system.
  * Implemented MapReduce-based analytics.
  * Implemented data validation toolset.
  * Fully automated DevOps infrastructure.

Technologies used in this project:

  * Linux
  * Ruby and CoffeeScript
  * [Riak](http://basho.com/riak/) - distributed key/value NoSQL database
  * [Couchbase](http://www.couchbase.com/) - document-oriented NoSQL database
  * [Ruby on Rails](http://rubyonrails.org/) - web-app framework
  * [jQuery Mobile](http://jquerymobile.com/) - HTML5-based UI system
  * [Unicorn](http://unicorn.bogomips.org/) and [Nginx](http://wiki.nginx.org/Main) - web servers
  * [Amazon Web Services (AWS)](http://aws.amazon.com/) - cloud computing
  * [Opscode Chef](http://www.opscode.com/) - infrastructure automation platform
  * [SendGrid](http://sendgrid.com/) - transactional emails
  * [CampaignMonitor](http://www.campaignmonitor.com/) - marketing emails
  * [Papertrail](https://papertrailapp.com/) - centralized logging
  * [Airbrake](http://airbrake.io/) - error monitoring (Ruby)
  * [NewRelic](http://newrelic.com/) - application performance and server monitoring

Check out more [screenshots](http://www.behance.net/gallery/VERA-Mobile-Web-App/9226521) on Leif's Behance page.
