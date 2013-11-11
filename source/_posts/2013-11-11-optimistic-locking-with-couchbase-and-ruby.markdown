---
layout: post
title: "Optimistic Locking With Couchbase and Ruby"
date: 2013-11-11 10:35
comments: true
categories:
- ruby
- couchbase
- nosql
- concurrency
- distributed systems
---

Concurrent modification of shared data can be a problem in any distributed system regardless of what data store you are using. With ACID-compliant relational databases, a common tactic is to use pessimistic locking at the table or row level. Most NoSQL data stores do not have a pessimistic lock operation, and even when they do, it is often considered a performance hazard. So, most applications do not lock objects before writing them to a NoSQL datastore (or they use an external lock of some sort). This can quickly become a problem when you have a distributed system with write contention, as shown in the figure below:

{% img center /images/posts/2013-11-11-optimistic-locking-with-couchbase-and-ruby/wrong.png %}

One of the nice features of Couchbase is its "CAS" operation. This provides the ability to do an atomic check-and-set operation. You can set the value of a key, providing the last known version identifier (called a "CAS value"). The write will succeed if the document has not been modified since you read it, or it will fail if it has been modified and now has a different CAS value.

Using this operation, we can easily build a higher-level operation to provide [optimistic locking](http://en.wikipedia.org/wiki/Optimistic_concurrency_control) on our documents, using a CAS retry loop. The idea is simple: get the latest version of the document, apply your update(s), and write it back to Couchbase. If there are no conflicts, then all is well, and you can move on. If there is a conflict, you _re-get_ the latest version of the document, _fully reapply_ your modifications, and try again to write the document back to Couchbase. Repeat until the write succeeds.

<!-- MORE -->

With this, the figure above would look like this:

{% img center /images/posts/2013-11-11-optimistic-locking-with-couchbase-and-ruby/right.png %}

There are a few things that are important to note about this technique.

1. You should have no unsaved modifications to the document before doing this. They will be lost.
2. Your modification code _must_ be re-runnable and not have undesired side-effects, because it may be run an unpredictable number of times.
3. Your modification code _should_ be [commutative](http://en.wikipedia.org/wiki/Commutative_property), since multiple clients may be operating at the same time, and we cannot guarantee order.
4. If your modification is not commutative, you should be comfortable that this roughly amounts to a Last Writer Wins (LWW) strategy (although that is not strictly guaranteed without a real vector clock).

## Example Code

I have created a [GitHub repository](https://github.com/scottwb/couchbase-optimistic-locking) that implements this technique by extending the Couchbase Ruby client's `Couchbase::Bucket` class on which you normally call `get` and `set` methods. You can, of course, put this elsewhere so that you don't need to monkey-patch someone else's library. Here is a look at the code:

``` ruby
require 'couchbase'

module Couchbase
  class Bucket
    # IMPORTANT:
    #   This method assumes that the doc you pass in is unmodified.
    #   Any unsaved changes to it will be discarded.
    #
    #   It loads a clean copy of the doc and passes it to the given
    #   block, which should apply changes to that document that can
    #   be retried from scratch multiple times until they are successful.
    #
    #   This method will return the final state of the saved doc.
    #   The caller should use this afterward, instead of the object it has
    #   passed in to the method call.
    #
    def update_with_retry(key, doc, &block)
      begin
        doc, flags, cas = get(key, :extended => true)
        yield doc
        set(key, doc, :cas => cas)
      rescue Couchbase::Error::KeyExists
        retry
      end
      doc
    end
  end
end
```

With this monkey-patch loaded, you can now do the following:

``` ruby
cb = Couchbase.connect(:bucket => 'my_bucket', :hostname => 'localhost')
doc = cb.get('some-key')

doc = cb.update_with_retry('some-key', doc) do |d|
  # Modify `d` to your heart's content
end

# Now doc has safely been updated by the block and saved without collision.
```

It is important to note that, if your changes are not commutative, like our simple increment example, the code in your modification block will probably want to be smart enough to do some kind of merge logic for conflict resolution. It must recognize that the state of the document before calling `update_with_retry` may not actually be the same state that the successful block operates on.

[Test code for this method](https://github.com/scottwb/couchbase-optimistic-locking/blob/master/spec/lib/couchbase_bucket_spec.rb) can be seen in the GitHub repository.

**Also note:** My college [Jeremy Groh](www.linkedin.com/in/jgroh9/) has a similar post with sample code for doing [optimistic locking on Couchbase using C#](http://www.ramsmusings.com/2013/06/11/optimistic-locking-with-couchbase/).
