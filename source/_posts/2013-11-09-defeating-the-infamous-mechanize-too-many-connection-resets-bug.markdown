---
layout: post
title: "Defeating The Infamous Mechanize \"Too Many Connection Resets\" Bug"
date: 2013-11-09 07:56
comments: true
categories:
- ruby
- bugs
- hacks
- mechanize
---

Have you ever seen this nasty, obnoxious error when using the Mechanize gem to write a screen scraper in Ruby?

```
Net::HTTP::Persistent::Error: too many connection resets (due to Connection reset by peer - Errno::ECONNRESET) after 2 requests on 14759220
```

This has plagued Mechanize users for years, and it's never been properly fixed. There are a lot of voodoo suggestions and incantations rumored to address this, but none of them seem to really work. You can read all about it on [Mechanize Issue #123](https://github.com/sparklemotion/mechanize/issues/123).

I believe the root cause is how the underlying Net::HTTP handles reusing persistent connections after a POST -- and there is some evidence on the aforementioned github issue that supports this theory. Based on that assumption, I crafted a solution that has been working 100% of the time for me in production for a few months now.

<!-- MORE -->

## The Workaround

This is not really a fix for Mechanize or Net::HTTP::Persistent, and there are sure to be some corner cases where you legitimately want this error to be bubbled up, but in practice, I have found that simply handling a persistent connection being reset with the "too many connection resets" error, _forcing the connection to be shutdown and recreated_, and simply trying again has worked 100% of the time in high-volume production for scrapers that suffered this problem intermittently.

This is done by creating a wrapper for `Mechanize::HTTP::Agent#fetch`, the low level HTTP request method that is used to do GETs, PUTs, POSTs, HEADs, etc. This wrapper catches this annoying little exception, and uses the `shutdown` method to effectively create a new HTTP connection, and then tries the `fetch` again.

Loading the following monkey-patch somewhere in your application ought to shutup this annoying error for you for most use cases:

```ruby
class Mechanize::HTTP::Agent
  MAX_RESET_RETRIES = 10

  # We need to replace the core Mechanize HTTP method:
  #
  #   Mechanize::HTTP::Agent#fetch
  #
  # with a wrapper that handles the infamous "too many connection resets"
  # Mechanize bug that is described here:
  #
  #   https://github.com/sparklemotion/mechanize/issues/123
  #
  # The wrapper shuts down the persistent HTTP connection when it fails with
  # this error, and simply tries again. In practice, this only ever needs to
  # be retried once, but I am going to let it retry a few times
  # (MAX_RESET_RETRIES), just in case.
  #
  def fetch_with_retry(
    uri,
    method    = :get,
    headers   = {},
    params    = [],
    referer   = current_page,
    redirects = 0
  )
    action      = "#{method.to_s.upcase} #{uri.to_s}"
    retry_count = 0

    begin
      fetch_without_retry(uri, method, headers, params, referer, redirects)
    rescue Net::HTTP::Persistent::Error => e
      # Pass on any other type of error.
      raise unless e.message =~ /too many connection resets/

      # Pass on the error if we've tried too many times.
      if retry_count >= MAX_RESET_RETRIES
        puts "**** WARN: Mechanize retried connection reset #{MAX_RESET_RETRIES} times and never succeeded: #{action}"
        raise
      end

      # Otherwise, shutdown the persistent HTTP connection and try again.
      puts "**** WARN: Mechanize retrying connection reset error: #{action}"
      retry_count += 1
      self.http.shutdown
      retry
    end
  end

  # Alias so #fetch actually uses our new #fetch_with_retry to wrap the
  # old one aliased as #fetch_without_retry.
  alias_method :fetch_without_retry, :fetch
  alias_method :fetch, :fetch_with_retry
end
```
