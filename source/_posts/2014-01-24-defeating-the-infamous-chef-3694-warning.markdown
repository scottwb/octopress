---
layout: post
title: "Defeating The Infamous CHEF-3694 Warning"
date: 2014-01-24 19:47
comments: true
categories:
- chef
- hacks
- bugs
- ruby
---

_**TL;DR:** I hate the CHEF-3694 warning, so I made a [cookbook](https://github.com/facetdigital/chef_resource_merging) to get rid of it. YMMV._

Resource cloning in Chef is a bit of a minefield. They have a ticket known as [CHEF-3694](https://tickets.opscode.com/browse/CHEF-3694) saying that the feature should be removed, and indicating that it will be by the time Chef 12.0.0 comes out. However, a lot of their Opscode-developed community cookbooks use (abuse?) resource cloning. The result is that you get tons of warnings about resource cloning that look like this:

```
[2014-01-24T16:15:55+00:00] WARN: Cloning resource attributes for package[perl] from prior resource (CHEF-3694)
[2014-01-24T16:15:55+00:00] WARN: Previous package[perl]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/perl/recipes/default.rb:26:in `block in from_file'
[2014-01-24T16:15:55+00:00] WARN: Current  package[perl]: /tmp/vagrant-chef-1/chef-solo-1/cookbooks/iptables/recipes/default.rb:21:in `from_file'
```

Where I come from, it's considered an error to have a warning in your output. Ignorable warnings bury important ones. So...for better or worse, I embarked upon a journey to see what I could do to use resources correctly and avoid these warnings...

<!-- MORE -->

## What is resource cloning and why are you warning me about it?

The [discussion](https://tickets.opscode.com/browse/CHEF-3694) about this this issue is an interesting read. You should be able to, for example, declare a service resource in one spot of your recipe, and later start it. You should also be able to have multiple recipes be able to install the same package resource and have it be idempotent, without having to worry about coordinating between cookbooks. That's Chef's job. To support this, Chef uses a technique they call _resource cloning_, which spews out warning messages because they plan to get rid of it. Proponents of the warning messages argue that if your cookbook relies on resource cloning, then you are doing something incorrectly and you have bigger problems. However, there are popular community cookbooks that won't work without it.

Here's an example of stock `perl` and `iptables` cookbooks causing this problem:

[{% img center /images/posts/2014-01-24-defeating-the-infamous-chef-3694-warning/CHEF-3694-perl.png %}](/images/posts/2014-01-24-defeating-the-infamous-chef-3694-warning/CHEF-3694-perl.png)

I really wouldn't want `perl` and `iptables` to have to coordinate between each other in order to avoid this warning. Perhaps there is a way to re-order them? Not that I could figure out...at least not without either making dangerous assumptions or editing stock community cookbook code.

Even so...there are cookbooks that have this problem by themselves without the help of other cookbooks. For example, one of the most popular cookbooks, `apache2`:

[{% img center /images/posts/2014-01-24-defeating-the-infamous-chef-3694-warning/CHEF-3694-apache.png %}](/images/posts/2014-01-24-defeating-the-infamous-chef-3694-warning/CHEF-3694-apache.png)

## Can we just remove resource cloning?

Since it was well-argued that cookbooks shouldn't rely on resource cloning, and that it would be removed in a future version of Chef, I decided to replace it myself with resource _duplication_. Resource cloning and its associated warning messages are handled in a method called `Chef::Resource::load_prior_resources`, so I just monkey-patched out that method to allow the duplicate resource without copying over any of the existing resources's attributes, using a bit of code like this:

``` ruby
class Chef
  class Resource
    def load_prior_resource
      Chef::Log.warn("I AIN'T CLONING #{self.to_s}!!!")
      true
    end
  end
end
```

NOPE! That doesn't work. While this works for some of my cookbooks and certain resources, the community `apache2` recipes clearly rely on the soon-to-be-deprecated resource cloning behavior. These recipes define the `service[apache2]` resource a number of times to do things like enable/start/restart after config changes. Without resource cloning, the `apache2::logrotate` recipe, for example, fails to process a restart of the apache2 service because it _didn't_ inherit the necessary attributes that needed to be cloned from the original service definition, giving errors like this:

```
================================================================================
Error executing action `restart` on resource 'service[apache2]'
================================================================================


Chef::Exceptions::Service
-------------------------
service[apache2]: unable to locate the init.d script!


Resource Declaration:
---------------------
# In /tmp/vagrant-chef-1/chef-solo-1/cookbooks/apache2/recipes/logrotate.rb

 20: apache_service = service 'apache2' do
 21:   action :nothing
 22: end
 23:



Compiled Resource:
------------------
# Declared in /tmp/vagrant-chef-1/chef-solo-1/cookbooks/apache2/recipes/logrotate.rb:20:in `from_file'

service("apache2") do
  action [:nothing]
  supports {:restart=>false, :reload=>false, :status=>true}
  retries 0
  retry_delay 2
  service_name "apache2"
  pattern "apache2"
  startup_type :automatic
  cookbook_name :apache2
  recipe_name "logrotate"
end
```

## Then how about reusing the existing resource?

I think resource reuse is probably the intention in 99% of the use cases. Some commenters on this discussion have suggested making all their recipes look up the resource in the _resources collection_ first, and using the existing one if possible, otherwise handling the not-found exception and creating the new resource. Not a bad suggestion...but there's no way I'm going to modify every community cookbook to do that.

As an experiment, I tried simply overriding the `service` DSL method (which is actually implemented in `method_missing`) to test this theory, with some monkey-patching like this:

``` ruby
class Chef
  module DSL
    module Recipe
      def service(svc, &block)
        s = run_context.resource_collection.find("service[#{svc}]")
        s.instance_eval(&block) if block
        s
      rescue Chef::Exceptions::ResourceNotFound => e
        method_missing("service", svc, &block)
      end
    end
  end
end
```

That's close, but it doesn't quite work. The most noticeable failure with this is that only the last `action` will be run. So for example, say you have something like this:

``` ruby
service "apache2" do
  action :enable
end

# ...some other stuff...

service "apache2" do
  action :start
end
```

Normally, this creates two `service[apache2]` resources, each copying its configuration from the previous definition, and _overriding_ the action(s). When executed, you'd end up with both actions being executed (but with a bunch of warnings that you're using the dreaded resource cloning).

With the reuse technique above, the problem is that, in this simple example, the `action: start` _overwrites_ the `action: enable`. In the end, you have your service started...but `chkconfig` shows that it was never enabled. This can obviously be much worse in more complex scenarios.

## The Workaround: resource merging

My workaround for this takes advantage of internal knowledge of how the `action` DSL method works...and it only applies to that one method. We're in dark magic territory, so I am sure this could potentially break somebody's cookbooks.

Building on the resource reuse attempt above, I made it so that instead of letting the `action` of a resource stomp over the pre-existing resource's action, it would _merge_ the actions together. In the over-simplified version, this looks like replacing the single `instance_eval` line from above with code like this:

``` ruby
combined_actions = s.action
if block
  s.instance_eval(&block)
  combined_actions += s.action
end
s.action combined_actions
```

## Putting it together

There are a few details I glossed over, such as managing the `:nothing` action, different default actions for different types of resources, actions that are Arrays vs Symbols, etc. My final solution was to extend `Chef::DSL::Recipe` with a `reusable_resource` method that could be used by specific resource DSL overrides as much or as little as you want. Here's what that looks like:

``` ruby
class Chef
  module DSL
    module Recipe
      def reusable_resource(
        resource_type,
        resource_name,
        default_action,
        &block
      )
        resource_str = "#{resource_type}[#{resource_name}]"
        existing_resource = run_context.resource_collection.find(resource_str)
        actions_before = existing_resource.action
        actions_before = [actions_before] unless actions_before.is_a? Array
        if block
          existing_resource.instance_eval(&block)
          actions_after = existing_resource.action
        else
          actions_after = []
        end
        if actions_after.nil? || actions_after.empty?
          actions_after = [default_action]
        end
        combined_actions = actions_before + actions_after
        combined_actions.delete(:nothing) if combined_actions.count > 1
        existing_resource.action combined_actions
        existing_resource
      rescue Chef::Exceptions::ResourceNotFound => e
        method_missing(resource_type, resource_name, &block)
      end
    end
  end
end
```

With that, if you only wanted to override the default behavior for `package` and `service` resources, you could monkey-patch those in like this:

``` ruby
class Chef
  module DSL
    module Recipe
      def reusable_resource
        # Omitted for brevity
      end

      def package(pkg, &block)
        reusable_resource("package", pkg, :install, &block)
      end

      def service(svc, &block)
        reusable_resource("service", svc, :nothing, &block)
      end

    end
  end
end
```

Now all those warnings are gone. My complete initial install works great without complaint. So do my subsequent re-runs.

I've packaged this all up as a cookbook that has nothing but a library applying these monkey-patches. You can [grab it from GitHub](https://github.com/facetdigital/chef_resource_merging) and put it at the front of your run_list with `recipe[chef_resource_merging]`.

## Limitations

This technique will probably fail in scenarios where you want to have multiple resources with the same name that have differing attributes other than `action`. For example, two different `bash` resources in two different places, with two different `command` scripts, with the same name. Either resource cloning or resource duplication would work...but resource merging the way I've implemented it is going to crash and burn. Of course you can simply name these resources differently, but given that resources share a global namespace, there's always a risk unless you make sure to prefix your resource names with something uniquely yours.

This is why I factored this technique into a `reusable_resource` DSL method. You can use it directly in custom cookbooks if you want. You can override specific types of resources as I have shown in the example, only touching `package` and `service`. Or, you can override those with additional logic to only do in in narrower cases (e.g., only if there is no block given). That's up to you. Your Mileage May Vary.

## Discussion

I welcome any and all discussion on this. Especially from someone who knows the internals of Chef much more deeply than I do, who can tell me if I'm getting myself into too much trouble here.

I'm hoping that some day there is a proper mechanism for resource reuse, when that is what is intended, or perhaps some way to detect if two resources internals are the same and make a smart decision about whether to reuse or duplicate. Maybe a real resource merging solution could happen, where the entire blocks are chained and executed? Or perhaps we'll see some resource namespace solution (though that would not have solved any of the issues I've had).
