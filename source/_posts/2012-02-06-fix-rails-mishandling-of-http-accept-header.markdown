---
layout: post
title: "Fix Rails Mishandling of HTTP Accept Header"
date: 2012-02-06 13:15
comments: true
categories: 
- rails
- bugs
- hacks
---

There are two Rails issues in its handling of the HTTP `Accept` header
which cause spurious `ActionView::MissingTemplate` exceptions.
This can be annoying when you are receiving emails on these via [Airbrake](http://airbrakeapp.com/),
especially given that most of these come from web spiders. I am encountering
this on Rails 3.0.7. One of these is fixed in a later version of Rails, but
for other reasons I can't upgrade right now. The other bug is still present in
Rails 3.2 and in master at the time of this writing.

I have worked up a gist with monkey-patches that fix both issues:

[https://gist.github.com/1754727](https://gist.github.com/1754727)

Read on to learn more about these issues and how my patches work...

<!-- MORE -->

## Rails Issue #736

[Issue #736](https://github.com/rails/rails/issues/736) is that Rails does not correctly parse a q-value in an Accept header when there is only one content-type specified. For example:

    Accept: text/html;q=0.9

Will likely cause a `MissingTemplate` exception because the request will not properly set `formats=[:html]`. ***This bug has existed since at least Rails 2.3, and still exists in Rails 3.2 and in master as of this writing.***

## Rails Issue #860

[Issue #860](https://github.com/rails/rails/issues/860) is that Rails does not properly parse wildcard content-types such as `text/*`, `application/*`, and `image/*`. For example:

    Accept: text/*

will likely cause a `MissingTemplate` exception because the request will not properly set `formats[:html,:text,:js,:css,:ics,:csv,:xml,:yaml:json]`. ***This bug was fixed sometime later than Rails 3.0.7.*** That fix, however, only solved the cases for `text/*` and `application/*`. The monkey-patch for this is largely based off of their solution, but also includes a fix for `image/*`.

## Testing This Patch

[The gist](https://gist.github.com/1754727) includes a file named `mime_type_spec.rb`, which is an RSpec example that you can drop into your rspec directory somewhere and run with the rest of your tests. It tests for both issues. Try it before dropping in the other files and see the failure. Then, add in the other files and see the tests pass.

When you change to a different version of Rails, the patches will be disabled, and the tests may fail. If they do, edit the patches to change the Rails version they are constrained to, and see if the tests pass. If not, you may need to do more investigation as the internals of Rails may have changed. On the other hand, if you upgrade to a new version of Rails and the tests still pass, then this means the bugs have been fixed in your version of Rails and you can delete these patches.

## Applying This Patch

[The gist](https://gist.github.com/1754727) contains files named `rails_issue_736.rb` and `rails_issue_860.rb`. These are the monkey-patches for these two issues, respectively. Place them somewhere in your Rails project where they will be loaded. I put them in `config/initializers`, but you should be able to put them anywhere that will get them loaded in your project.

You can take one or both files at your discretion.

You'll notice at the top of each of these is a test against the version of Rails you are running, that looks like:

``` ruby
if Rails.version == "3.0.7"
```

You will want to edit this to be equal to your version of Rails. This makes it so that the monkey-patch gets applied, but only for your current version of Rails. This way, when you upgrade Rails, the patch is disabled. Then, if your tests no longer fail, you know Rails has fixed it and you can delete this monkey-patch. Otherwise, you can go back and edit that version number again and see if it makes the tests pass again.

## The Future

In my opinion, some simple refactoring of `Mime::Types.parse` would eliminate the maintenance hazard that causes these issues. Currently there are two blocks of code in this method: one that handles a single content-type, and one that handles a list of multiple content-types. This should really just treat the single case as a list of one content-type. This way, fixes made to one don't have to be applied to the other -- which is likely the main cause of these problems. Both these issues are non-issues in the multiple content-type case, but are not addressed in the single content-type case.

I'll probably work up a pull request to address this, so hopefully these will be fixed in the mainline Rails someday.
