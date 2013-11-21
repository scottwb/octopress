---
layout: post
title: "Cohort Analysis"
date: 2013-01-21 13:58
comments: true
categories:
- lean startup
- analytics
- metrics
---
After a few conversations with a former colleauge about statistics, metrics, and analytics, I thought I'd dig up some good articles to summarize some of the key Lean Startup principles surrounding metrics and data-driven decision-making for those who haven't had the pleasure of reading Eric Reis's book _Lean Startup_. That turned into a blog post about what metrics we report as developers and ops guys, and why we need to focus on _Actionable Metrics_ over _Vanity Metrics_.

 My goal is to provide a little background on _Actionable Metrics_ and how they differ from _Vanity Metrics_. Understanding this is a central theme of the book _Lean Startup_, by Eric Reis, and the writings and teachings of a number of other prominent startup/entrepreneur/lean proponents such as Steve Blank, Dave McClure, and Ash Maurya. Fred Wilson of Union Square Ventures, has been quoted saying, "one of our firm's favorite measurements is the cohort analysis".

<!-- MORE -->

## Three Links You Should Read

### 1. [3 Rulesource/_includes/custom/asides/s To Actionable Metrics - Ash Maurya](http://www.ashmaurya.com/2010/07/3-rules-to-actionable-metrics/)
In this post, Ash Maurya, author of _Running Lean_ and creator of _Lean Canvas_ starts right out with the definitions of Actionable Metrics and Vanity Metrics. The key summary being:

> **Actionable Metric:** ties specific and repeatable actions to observed results
>
> **Vanity Metric:** only serves to document the current state of the product but offers no insight into how we got here or what to do next.

In the _Tracking Long LifeCycle Events_ section of this post, where Maurya talks about cohort analysis, his recommendation is that the first report you implement -- the canary in the coal mine -- is exactly the kind of reporting we put first on our internal statistics console:

> The first report I recommend implementing is a “Weekly Cohort Report by Join Date”. This report functions like a canary in the coal mine and is a great alerting tool for picking up on actions that had overall positive or negative impact.


### 2. [Track What Matters With Cohort Analysis - Martin Thomas](http://www.purlem.com/blog/2012/03/track-what-matters-with-cohort-analysis/)
This is a nice short blog post, by Martin Thomas, founder of Purlem, about using cohort analysis with a simple real-world example. Importantly, he calls out the definition of a cohort:

> A cohort is a group of people who share a common characteristic or experience within a defined period

Note the focus on measuring "within a defined period". As with Maurya's post, his example is to group cohorts by signup date, and then track what those cohorts have as their initial experience (his is over a month, ours is currently over a 24-hour period -- we really want to measure that "Day One Aha!" experience). He quotes Eric Reis on the issue of using vanity metrics:

> Before using cohort analysis, I was tracking the cumulative number of paying users. Eric Reis calls this vanity metrics as they give the “rosiest possible picture” of a startup’s progress, but does not track how people are actually interacting with the application.
>
> At the end of the day, using cohort analysis helps you to track the numbers that matter to the progress of your company.


### 3. [Lean Startup Metrics & Analytics - Nocola Junior Vitto](http://www.slideshare.net/njvitto/lean-startup-metrics-analytics)
This is a great slide deck about metrics and analytics in a startup. It's a bit long but I think it is worth looking at the slides and understanding them. If you don't go through them all, at least check out what I consider to be the highlights:

* **Slides 4-8:** The perfect picture of vanity metrics. I love that Slide 7 calls Google Analytics Realtime Overview a "drug that can kill you". So true.

* **Slides 14-15:** Good definitions of actionable metrics

* **Slide 28:** Nice visualization of the conversion funnel

* **Slide 33:** Good progression of online marketing. We need to work toward getting solidly into the 3rd Generation territory.

* **Slides 49-56:** Good summary of metrics surrounding user acquisition. This is exactly why I've been so adamant about getting the people sharing links to use tracking codes properly: "Use **unique urls** (tracking parameters) on **every url** you create/give-out/pay for"

* **Slides 71-77:** Good overview of the "Viral Coefficient". In our app we track this using AddThis analytics (which is one of the reasons we chose to use AddThis instead of custom Facebook integration)

* **Slides 78-81:** Good overview of the "Net Promoter Score" (NPS). We intend to measure this using Qualaroo (formerly KISSinsights), but we are not doing that yet.

* **Slide 98:** I love this slide -- it applies the "OODA Loop" to the Lean Startup. [The OODA Loop](http://en.wikipedia.org/wiki/OODA_loop) is a term uses in martial arts, combatics, military, and law-enforcement. It stands for _Observe, Orient, Decide, Act_. Then repeat. Eric Reis talks about the "Build-Measure-Learn" cycle. They really are the same thing. I love the idea of applying the OODA Loop to startups because it imbues a sense of urgency. This slide has a nice picture merging the two.

* **Slides 99-100:** Innovation Accounting. I'd call it a wake-up-call. "Everything you do should attempt to change a metric". I.e.: anything we are not doing that is not specifically aimed at changing an actionable metric is something we need to stop doing.

* **Slide 106:** Kanban board. Looks like my Trello boards!
