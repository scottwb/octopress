---
layout: post
title: "Compile A Single CoffeeScript File From Your Rails Project"
date: 2012-06-30 20:56
comments: true
categories: 
- rails
- rake
- ruby
- coffeescript
---

Using Rails 3.2 (or 3.1 for that matter), you typically have CoffeeScript (and a javascript engine like therubyracer/V8) installed as gems - probably via Bundler from your Gemfile, possibly in a per-project RVM gemset.

It doesn't seem that this type of setup includes direct access to the `coffee` command-line compiler. Instead the `coffee-script` gem uses calls directly to the compiler. Sometimes I'd just like to compile one of the CoffeeScript files in my Rails app just to look at the javascript, or maybe to export it somewhere that I'm not using CoffeeScript.

To do this, I've made a simple Rake task. Just drop this into `lib/tasks/coffee.rake` in your Rails project:

``` ruby coffee.rake https://gist.github.com/3026787 View Gist
namespace :coffee do
  task :compile, :filename do |t, args|
    filename = args.filename
    puts CoffeeScript.compile(File.open(filename))
  end
end
```

Then, you can compile a single file from like this:

```bash
$ rake coffee:compile[app/assets/javascripts/mystuff.js.coffee]
```

This will output it to stdout so you can redirect that to a file, or the `gist` command, or whatever.
