---
layout: post
title: "Customize Attribute Accessors on Ripple Models"
date: 2012-07-12 09:18
comments: true
categories:
- ripple
- riak
- rails
- ruby
---

Sometimes in your Ruby on Rails models, you want to override the attribute accessors to do additional processing on the values on their way in and out of the data storage layer. Custom serialization is a common use of this.

For example, when using ActiveRecord for your Rails models, you can provide custom attribute accessors, say to serialize a Hash to JSON, using the `read_attribute` and `write_attribute` methods like this:

```ruby
# Assume a text column named 'stuff'
class User < ActiveRecord::Base
  def stuff
    JSON.parse(read_attribute(:stuff))
  end

  def stuff=(new_val)
    write_attribute(:stuff, new_val.json)
  end
end
```

With this, you can assign a Hash to the `stuff` attribute of `User`, and when you access it via `User#stuff`, you get a Hash back. All the while, it's read and written to and from the database as a JSON string.

### Doing it with Ripple on top of Riak

[Ripple](https://github.com/seancribbs/ripple/) is the Ruby modeling layer for the distributed NoSQL store, [Riak](http://basho.com/products/riak-overview/). It tries very hard to provide a lot of the same interfaces as ActiveRecord. However, this is one of the areas it diverges: `Ripple::Document` objects do not support the `read_attribute` and `write_attribute` methods.

Instead, they implement the `[]` and `[]=` methods. Translating the code above to work with Ripple is pretty easy:

```ruby
class User
  include Ripple::Document

  property stuff, Hash

  def stuff
    JSON.parse(self[:stuff])
  end

  def stuff=(new_val)
    self[:stuff] = new_val.to_json
  end
end
```

### Bonus Points

Using this tactic, you can easily add some memoization so that your getter doesn't need to parse the JSON text on every access. To do this, we'll use an instance variable as a cache that we'll invalidate in the setter, like so:

```ruby
class User
  include Ripple::Document

  property stuff, Hash

  def stuff
    @cached_stuff ||= JSON.parse(self[:stuff])
  end

  def stuff=(new_val)
    @cached_stuff = nil
    self[:stuff] = new_val.to_json
  end
end
```
