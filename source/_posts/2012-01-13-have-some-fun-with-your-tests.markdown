---
layout: post
title: "Have Some Fun With Your Tests"
date: 2012-01-13 14:36
comments: true
categories: rspec, development
---

I just came across this test case in some old code I had forgotten about:

``` ruby
it "validates that the address is geocodable" do
  @user.address = "Death Star, Suite 1138"
  @user.city    = "Moon System"
  @user.state   = "Endor"
  @user.zipcode = "1138"

  expect{@user.save!}.to raise_exception ActiveRecord::RecordInvalid
  @wall.errors[:address].should include "could not be found on the map"
end
```

It reminded me of something I actually think is important: have a little fun with the examples in your test cases. Not so much as to be distracting, but a little humor can add some fun to an otherwise mechanical, boring task.

I like to think of the guy who comes along a year later trying to understand my code by reading the tests. Telling a small story or making some pop-culture references will hopefully brighten his task.
