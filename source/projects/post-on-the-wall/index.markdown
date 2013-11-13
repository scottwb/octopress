---
layout: page
#title: "Post On The Wall"
#date: 2013-11-11 15:31
favicon: /projects/post-on-the-wall/favicon.png
comments: false
sharing: false
footer: false
---

#### Post On The Wall&trade; (2011)

{% img left /projects/post-on-the-wall/iphone-splash.png 'Post On The Wall' 'Post On The Wall' %}

Post On The Wall&trade; was a desktop web app and native iOS app that provided "virtual walls" at geographic locations, enabling users within a close range to upload pictures, sketches, and other content from their mobile devices, that were projected to a big screen with a real-time, animated view of the wall as it's content unfolded.

Unlike most social or photo-sharing apps, privacy in Post On The Wall was enforced by GPS location. In order to post a picture on a wall, you had to _be there_, and only people who were there could see the content unless it was otherwise shared on social networks. The map features allowed event-goers to find walls nearby and in the future, in order to discover events, see publicized photos, and register to attend.

{% img center /projects/post-on-the-wall/map.png 'Find a Wall' 'Find a Wall' %}

Our primary market focus was on event planners, wedding coordinators, and a variety of charity fundraising galas. The goal was to let event coordinators create these virtual photo walls where only their attendees were allowed to submit content. The event coordinator could sign up and create their own walls, choose from a variety of themes, and eventually charge a fee to allow posting to a live wall - a feature that was heavily requested at high-profile events - or even ordering high-resolution prints.

{% img center /projects/post-on-the-wall/create.png 'Create a Wall' 'Create a Wall' %}

At the event, a large screen TV or projector would display the wall in "presentation mode", which looked like a fullscreen wall texture with new posts sticking onto the wall in real time.

{% img center /projects/post-on-the-wall/presentation.png 'Presentation Mode' 'Presentation Mode' %}

When idle, the wall would animate, panning around and zooming in on individual posts to show the full resolution media, along with the comments, tags, signatures, and sketches submitted by the poster. It was an incredibly fun and addictive experience for those who participated.

{% img center /projects/post-on-the-wall/popup.png 'Viewing a Post' 'Viewing a Post' %}

<span class="finePrint">Images by <a href="http://www.placeit.net/" target="_blank">Placeit</a></span>

## My Contribution

As the CTO at Be Labs (formerly Balassanian Enterprises, LLC), I was responsible for setting the technology direction of a number of portfolio projects, including Post On The Wall. I oversaw the software development team that implemented the entire application from the backend image upload-processing web service, to the responsive-design web app and the native iOS app. Being a small team, I was also a significant individual contributor.

My biggest areas of individual contribution in software development included:

  * Scalable cloud-based architecture for elastically tuning compute resource provisioning for image upload processing based on live event load. Our first major event handled hundreds of posts per minute without a hitch.
  * Custom queue and background worker mechanism for allowing intermittently-connected iOS devices to asynchronously complete their upload tasks.
  * Designed and implemented a photo-placement algorithm that balanced efficient use of space (bin packing) and aesthetics.
  * Built the live, animated presentation mode front-end.
  * Implemented authentication for wall owners, and location-based permission for attendees.
  * Involved in the full stack implementation of the web app - Rails, HTML5, CSS, Javascript, etc.
  * Helped design the API interface between the backend and the iOS App.
  * Fully automated DevOps infrastructure.

Technologies used in this project:

  * Linux
  * Ruby, JavaScript, and Objective-C
  * [MySQL](http://www.mysql.com/) - relational database
  * [Ruby on Rails](http://rubyonrails.org/) - web-app framework
  * [Memcached](http://memcached.org/) - distributed memory cache
  * [jQuery Mobile](http://jquerymobile.com/) - HTML5-based UI system
  * [Google Maps API](https://developers.google.com/maps/) - custom dynamic mapping
  * [AddThis] - social sharing platform
  * [Unicorn](http://unicorn.bogomips.org/) and [Nginx](http://wiki.nginx.org/Main) - web servers
  * [Rackspace Cloud](http://www.rackspace.com/) - cloud computing
  * [Rackspace Cloud Files](http://www.rackspace.com/cloud/files/) - content delivery network
  * [Opscode Chef](http://www.opscode.com/) - infrastructure automation platform
  * [SendGrid](http://sendgrid.com/) - transactional emails
  * [Airbrake](http://airbrake.io/) - error monitoring
  * [NewRelic](http://newrelic.com/) - application performance and server monitoring
  * [TestFlight](https://testflightapp.com/) - iOS beta testing
