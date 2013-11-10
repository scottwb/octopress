---
layout: page
#title: "VERA Platform API"
#date: 2013-11-10 08:00
comments: false
sharing: false
footer: false
---
<a href="http://www.validas.com/" target="_blank">{% img center /projects/vera-platform-api/hero.png 'VERA Platform API' 'VERA Platform API' %}</a>

The VERA Platform API at [Validas](http://www.validas.com) is a RESTful Hypermedia API that gives clients access to rich usage and account data for their customers' wireless contracts and devices. This includes voice, text, and data usage, premium features, taxes and fees, device info, upgrade eligibility and contract end dates, demographic user information, and access to a data-driven savings recommendations from an up-to-date inventory of plans and features across all major wireless carriers.

Our customers are companies that have contact with individual wireless customers, who want to provide a variety of value-added services based on the details of their wireless usage profile and spending habits. This ranges from large electronics retailers (online and brick-and-mortar), to major wireless carriers, plan-switching and savings brokers, handset buy-back and recycling providers, comprehensive utility companies, expense management organizations, insurance underwriters, and even small boutique [MVNOs](http://en.wikipedia.org/wiki/Mobile_virtual_network_operator) and resellers.

## My Contribution

As the VP of Engineering at this small company, I took the lead with [Jeremy Groh](http://linkedin.com/in/jgroh9) (CTO) and [Leif Jensen](http://www.linkedin.com/in/leifjensen) (Dir. of Product) in pivoting the company's focus, based on our measurements and market feedback, from a direct-to-consumer approach to becoming a SaaS API to deliver the power of our platform to larger companies that have better connections to consumers, and have a real opportunity to drive value from our technology.

I was involved in both leading the software engineering team and serving as an individual contributor to the implementation of this project. Given the technical nature of the product, I was also heavily involved in the market research and customer development efforts, inbound marketing and feature roadmap planning, and the financial modeling of operation cost projects and pricing models.

This was roughly a 6-month project from conception to first public launch, including all aspects of implementation and operating as a self-contained business unit.

My biggest areas of individual contribution in software development included:

  * Overhauling major pieces of the VERA Platform architecture to focus on scalability, modularity, and testability.
  * Iteratively replacing portions of a legacy system with more modern designs and a service-oriented architecture (SOA).
  * RESTful Hypermedia API design from the ground up.
  * API implementation.
  * API documentation.
  * DevOps infrastructure automation.

Technologies used in this project:

  * Windows and Linux
  * Ruby, C#, and .NET
  * [RabbitMQ](http://www.rabbitmq.com/) - messaging system
  * [Couchbase](http://www.couchbase.com/) - document-oriented NoSQL database
  * [Sinatra](http://www.sinatrarb.com/) - lightweight API server
  * [Unicorn](http://unicorn.bogomips.org/) and [Nginx](http://wiki.nginx.org/Main) - web servers
  * [Amazon Web Services (AWS)](http://aws.amazon.com/) - cloud computing
  * [Opscode Chef](http://www.opscode.com/) - infrastructure automation platform
  * [SendGrid](http://sendgrid.com/) - transactional emails
  * [CampaignMonitor](http://www.campaignmonitor.com/) - marketing emails
  * [Papertrail](https://papertrailapp.com/) - centralized logging
  * [Airbrake](http://airbrake.io/) - error monitoring (Ruby)
  * [Raygun](http://raygun.io/) - error monitoring (C#)
  * [NewRelic](http://newrelic.com/) - application performance and server monitoring
