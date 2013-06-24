---
layout: post
title: "Quick and Dirty CampaignMonitor Webhook Management"
date: 2013-06-24 16:33
comments: true
categories:
- campaignmonitor
- ruby
- tools
---
You've got a CampaignMonitor mailing list and you're using their API to add/update/remove subscribers as they sign up for your app and opt-in to your mailing list. Great. But you also want to know when they unsubscribe from your newsletter, or when an existing user subscribes via a separate newsletter form on your website, and be able to keep that information sync'd with your user database.

CampaignMonitor provides the ability to create [Webhooks](http://www.campaignmonitor.com/api/webhooks/) that will drive an HTTP POST callback to your app when subscribe/unsubscribe events happen. Once you dive into this, you'll realize that you also need a way to deploy and update your webhooks. They only allow you to do this through their API -- there is no GUI for it.

<!-- MORE -->

I threw together a quick-and-dirty Rakefile using the `createsend` gem. First make sure you have either done `gem install createsend` or have added the `createsend` gem to your Gemfile. Then, you can create a Rakefile that looks something like this:

``` ruby
API_KEY = 'your_secret_api_key_here'
LIST_ID = 'your_mailing_list_id_here'  # Get this from the "change type" page for the list
WEBHOOK_URL = 'http://example.com/path/to/your_webhook.json'

def campaign_monitor_list
  CreateSend::List.new({:api_key => API_KEY}, LIST_ID)
end

# NOTE: That I depend on :environment for all of these. That is to load the
#       Rails environment I use them in. You can change that and require 'createsend'
#       explicitly if you like.
namespace :campaign_monitor do
  namespace :webhooks do
    desc "List all the CampaignMonitor webhooks"
    task :list => :environment do
      puts campaign_monitor_list.webhooks.inspect
    end

    desc "Register all our CampaignMonitor webhooks"
    task :create => :environment do
      campaign_monitor_list.create_webhook(
        ["Subscribe", "Deactivate"],
        WEBHOOK_URL,
        'json'
      )
    end

    desc "Test all our CampaignMonitor webhooks"
    task :test => :environment do
      list = campaign_monitor_list
      list.webhooks.map do |hook|
        list.test_webhook(hook[:WebhookID])
      end
    end

    desc "Uninstall all our CampaignMonitor webhooks"
    task :clear => :environment do
      list = campaign_monitor_list
      list.webhooks.map do |hook|
        list.delete_webhook(hook[:WebhookID])
      end
    end
  end
end
```

With this saved as `campaign_monitor.rake` and loaded by rake, you will now have the following tasks you can integrate into your deployment system:

``` bash
% rake -T campaign_monitor:webhooks
rake campaign_monitor:webhooks:clear   # Uninstall all our CampaignMonitor webhooks
rake campaign_monitor:webhooks:create  # Register all our CampaignMonitor webhooks
rake campaign_monitor:webhooks:list    # List all the CampaignMonitor webhooks
rake campaign_monitor:webhooks:test    # Test all our CampaignMonitor webhooks
```
