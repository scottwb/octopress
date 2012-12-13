---
layout: post
title: "Middleman Deployment Rakefile"
date: 2012-02-24 09:30
comments: true
categories:
- ruby
- rake
- middleman
- deployment
- rsync
---

I started tinkering with [Middleman](http://middlemanapp.com/) the other day to build a small static website.

In the past, I've used [StaticMatic](http://staticmatic.rubyforge.org/), [Jekyll](http://jekyllrb.com/), or [Octopress](http://octopress.org/) for this sort of thing. Jekyll and Octopress are great for blogs. StaticMatic is great for prototyping a static site with Haml and Sass that you can easily migrate to your Rails app.

{% pullquote %}
I've been using Middleman for less than 24 hours at this point, but so far it seems great. One of the things I like about it is that it's actually serving your site dynamically with Sinatra, and comes out of the box with Haml, Sass, Compass, view helpers, lorem ipsum generators, CSS and Javascript minification, and more. Plus, you can throw in any other Rack middleware you like, for example, the Google Analytics middleware. While you're developing the site, you run the preview server and your changes are immediately visible without restarting it. The cool part is that when you deploy, it actually generates all the static pages by executing the Rack server and running requests through it. This means {" any custom middleware you add to the stack gets invoked for all the pages it generates."}
{% endpullquote %}

Middleman imposes very little structure. This is refreshing if you're just making a one- or two-page site. It lets you get started right away and build whatever organization you want for yourself. It doesn't tell you how to deploy your site. That's I'm here to do. :)

I took a little inspiration from Octopress's Rakefile and put together a simple Rakefile for use with Middleman that helps you deploy your static site via rsync. It also adds a few little wrapper tasks for things like remembering to add the `--clean` flag, and an Octopress-like `rake gen_deploy` task that regenerates and deploys in one step.

``` bash
$ rake -T
rake build       # Build the website from source
rake deploy      # Deploy website via rsync
rake gen_deploy  # Build and deploy website
rake preview     # Run the preview server at http://localhost:4567
```

Simply drop this Rakefile into the root of your Middleman project and edit the SSH variables at the top.

``` ruby Rakefile https://gist.github.com/1902178 View Gist
SSH_USER = 'root'
SSH_HOST = 'www.example.com'
SSH_DIR  = '/var/www/html/www.example.com'

desc "Build the website from source"
task :build do
  puts "## Building website"
  status = system("middleman build --clean")
  puts status ? "OK" : "FAILED"
end

desc "Run the preview server at http://localhost:4567"
task :preview do
  system("middleman server")
end

desc "Deploy website via rsync"
task :deploy do
  puts "## Deploying website via rsync to #{SSH_HOST}"
  status = system("rsync -avze 'ssh' --delete build/ #{SSH_USER}@#{SSH_HOST}:#{SSH_DIR}")
  puts status ? "OK" : "FAILED"
end

desc "Build and deploy website"
task :gen_deploy => [:build, :deploy] do
end
```
