---
layout: post
title: "Wake Surfing With RubyMotion"
date: 2014-02-09 13:53
comments: true
categories:
- ruby
- rubymotion
- ios
- facetdigital
---

[{% img left /images/posts/2014-02-09-wake-surfing-with-rubymotion/swell.jpg 350 'Supra Boats Swell System' 'Supra Boats Swell System' %}](https://itunes.apple.com/us/app/supra-boats-swell-surf-system/id804765775?mt=8)

A couple weeks ago, our first [Facet Digital](http://facetdigital.com/) client project to go public in 2014 was launched in the Apple App Store: [Supra Boats Swell Surf System](https://itunes.apple.com/us/app/supra-boats-swell-surf-system/id804765775?mt=8).

This is a multimedia catalog app designed for reps and resellers on the boat show floor, allowing them to show off their [Swell System](http://supraboats.com/swell/) for wake surfing. It features an interactive look at some of the unqiue features of this system, such as the ability to change the shape of the wave and even which side of the boat the wave is on! I'm no professional wake surfer, but just watching some of the videos embedded in this app make me anxious for summer to arrive...and to participate in some of these fun photo shoots with the great folks at Supra Boats for next year's app.

[Jeremy](http://ramsmusings.com/), [Leif](http://koolmanchu.com/), and I had a blast building this app. Part of that was because the content was so cool. A bigger part was because because we decided, for the first time, to build an iOS app using 100% RubyMotion, instead of going the traditional XCode and Objective-C route.

So, I thought I'd share a litte bit about why we enjoyed using RubyMotion...

<!-- MORE -->

## 7 Reasons We Liked RubyMotion

Most of the code we've written in the last 5 - 10 years has been in higher-level, programmer-pleasing, dynamic languages like Ruby, Python, and JavaScript. While I do have an extensive background in C and C++, sometimes I cringe at the thought of going back there. That's what Objective-C represents to me. Sure, it has message passing like Smalltalk, and the last few years of improvements like ARC and some of the excellent static analysis tools are great, but for my programmer productivity buck, Objective-C in all its verbosity doesn't even compare to the expressiveness of Ruby.

Plus, I hate XCode. I'm an Emacs, Vim, and zsh guy. Keep my hands on the keyboard.

Here are a few of the highlights we experienced:

### 1. The Language

If you ask me, the Objective-C syntax is ugly and cumbersome. Maybe that's just the view through my Ruby-colored glasses, but just compare this simple Objective-C code:

```objc
// Objective-C
UIView *view=[[UIView alloc] initWithFrame:CGRectMake(x, y, width, height)];
[view setBackgroundColor:[UIColor colorWithPatternImage:[UIImage imageNamed:@"a.png"]]];
```

to the equivalent RubyMotion code:

```ruby
# ruby
view = UIView.alloc.initWithFrame([[x, y], [width, height]])
view.backgroundColor = UIColor.colorWithPatternImage(UIImage.imageNamed('a.png'))
```

Or how about native types, list comprehension, and block syntax?

```objc
// Objective-C
NSMutableDictionary* d = [NSMutableDictionary dictionary];
for (int i = 0; i < 100; ++i) {
  NSNumber *v = @(i);
  d[[v stringValue]] = v;
}

__block int sum = 0;
[d enumerateKeysAndObjectsUserBlock:^(id key, id onj, BOOL *stop) {
  sum += [obj intValue];
}];
```

```ruby
# ruby
d = {}
(0..100).each { |i| d[i.to_s] = i }

sum = d.values.inject(0) { |memo, value| memo + value }
```

Even though RubyMotion is compiled to LLVM just like the Objective-C code is, I am sure one could make a performance argument in favor of Objective-C in examples like this. However, I prefer to optimize for developer productivity, time to market, and ease of correctness. The beauty of RubyMotion is that you can call anything natively written in Objective-C or C anyway, so you always have the option to factor out performance sensitive pieces if you feel it is necessary.


### 2. The Test Infrastructure

If you're used to TDD in a language like Ruby, using tools like RSpec, you'll find the iOS/Objective-C ecosystem lacking in this department. RubyMotion integrates Bacon, a mini RSpec of sorts, that provides a great DSL for unit and integration testing. For example:

```ruby
describe "Dealer" do
  before do
    @dealer = Dealer.new(40, -100, "Cool Dealer", "http://dealer.com/")
  end

  it ".closest_dealer_to finds the closest dealer to a given location" do
    closest_dealer = Dealer.closest_dealer_to(
      CLLocation.alloc.initWithLatitude(
        47.7,
        longitude: -122.2
      )
    )
    closest_dealer.is_a?(Dealer).should == true
    closest_dealer.title.should == "Active Watersports"
  end
end
```

Running these tests is as easy as running the `rake spec` command. This builds all your app and library code, yoru test code, downloads a test executable to the simulator, runs it, and prints typical RSpec-style output right to the terminal. The build/test cycle is almost identical to what you'd do for a Ruby on Rails app. You can even run the test suite on the device with `rake spec:device` and can filter to specific test cases when you want to focus on one part of the app.

### 3. The Workflow

The workflow is all based around `rake`. Everything is done via the command line. No XCode required. For an old-school guy like me, this is way it should be. My workflow goes like this:

  * Write code in Emacs
  * Run `rake spec` to build the test suite, download it to the simulator, and and run it, seeing the output in my terminal.
  * Run `rake` to build the app, downloaded it to the simulator, and run it.
  * Play with the app on simulator.
  * Use the REPL to tinker with layout, variables, etc. live on the simulator!
  * Repeat as necessary.
  * Run `rake device` to build the app for my target device, download it there, and run it, so I can play with it on a real iPhone or iPad.
  * Run `rake testflight notes="Added new features!"` to build an AdHoc distribution, upload it to TestFlight, and push it out to all of my beta-testers.
  * Run `rake archive` to build the whole submission to the app store.
  * Drag the `.ipa` that outputs into the Application Loader, and I'm off to the App Store!

Note the part about the REPL! This is one of my favorite parts. I can't stand working in a langauge without a good REPL. It's one of the reasons I love working in Ruby and Python. Not only does RubyMotion provide a nice REPL, it is run in the context of the running app. That means you can call methods on live objects, print out their values, etc. This can come in super handy when you just want to tweak the positioning of something until you like it, and then read out its final values.

You can also use `puts` to print to `stdout` right there in the REPL in your terminal, like you're used to with your usual Ruby apps.

### 4. The Integration

It's very cool that out of the box, RubyMotion comes with [TestFlight](http://testflightapp.com/) integration. That's hands-down the best way to get your pre-release app builds in the hands of beta-testers and stakeholders.

If you're not into Emacs (why wouldn't you be?), [RubyMine](http://www.jetbrains.com/ruby/) by JetBrains is a great IDE for RubyMotion (as well as Ruby on Rails). It integrates with auto-completion of the iOS SDK API, and the rake-baesd test runner too. And it has support for Emacs key-bindings. ;)

### 5. Great Libraries

You can't talk about RubyMotion without mentioning [BubbleWrap](https://github.com/rubymotion/BubbleWrap) and [SugarCube](https://github.com/rubymotion/sugarcube). Both of these libraries add tremendous value and easy-of-use to RubyMotion. They provide two different perspectives on wrapping CocoaTouch/iOS APIs with more idiomatic Ruby interfaces -- with some amount of overlap. These go a long way to making your code even less verbose -- particularly when dealing with the direction one-to-one nature of calling Objective-C APIs from RubyMotion.

Consider SugarCube, for example:

```ruby
view.on_swipe(direction: :left, fingers: 2) do |gesture|
  # handle two-finger left swipe
end
```

You don't even want to know what that code looks like in raw RubyMotion, let alone Objective-C. SugarCubes adds some useful tools to the REPL as well, like the `tree` command that outputs information about your view hierarchy, and lets you gain access to these easily:

```
(main)> tree
  0: . UIWindow(#6e1f950: [[0.0, 0.0], [320.0, 480.0]])
  1: `-- UIView(#8b203b0: [[0.0, 20.0], [320.0, 460.0]])
  2:     +-- UIButton(#d028de0: [[10.0, 10.0], [320.0, 463.400512695312]])
```

BubbleWrap provides a JSON interface that is identical to the native Ruby JSON API, making your web service consumers from your Rails apps easy to port over. It also gives us some great one-line high-level wrappers for common operations like playing a movie or taking a picture with the camera, for example:

```ruby
# Uses the rear camera
BW::Device.camera.rear.picture(media_types: [:movie, :image]) do |result|
  image_view = UIImageView.alloc.initWithImage(result[:original_image])
end
```

Before you re-invent anything in RubyMotion...look in BubbleWrap and SugarCube first. They're open source, of course, so even if you need to customize their behavior, you can learn a lot by looking at their source code.

### 6. Compiled Performance

Unlike other non-Objective-C iOS app frameworks like Sencha Touch or PhoneGap, RubyMotion compiles to the same LLVM code that Objective-C does. It uses the same compilers. The secret is that RubyMotion is not Ruby. It is a subset of Ruby, based on MacRuby. There are a few features of Ruby that do not exist, but most of them do. This is because Objective-C and Ruby are very similar, and both attribute their message-passing and object-oriented nature to a Smalltalk heritage.

You can program slick animations with RubyMotion that perform as well as those made in Objective-C, without dealing with the sub-par performance of the JavaScript engine running a bloated jQuery library inside a `UIWebView`, like you would with PhoneGap.

### 7. Memory Management

Memory Management - you don't have to. The RubyMotion team has basically built their own ARC-style memory management baked into the language constructs. You use Ruby like you always do, the RubyMotion framework takes care of auto-releasing unreferenced objects, similar to Apple's ARC. You rarely need to worry about references, but in the rare case that you do (e.g., for cyclical references), the [WeakRef](http://www.rubymotion.com/developer-center/guides/runtime/#_weak_references) class gives you what you need. The RubyMotion docs are pretty good too, and they generally point out very explicitly when you are in danger of needing to use a `WeakRef`.

## Conclusion

Developing an iOS app with RubyMotion was fast and fun. I believe programming should be fun as much as possible. Anything tedious should be automated or abstracted -- and that's exactly what RubyMotion has done for iOS programming. We'll definitely be using RubyMotion at [Facet Digital](http://facetdigital.com/) for a few more iOS apps coming out in the next few months...
