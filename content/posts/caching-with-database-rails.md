---
title: "Rails - Caching with the database"
date: 2019-11-15T14:36:17-06:00
draft: false
description: "A brief description of Hugo Shortcodes"
categories: [
  "rails"
]
tags: [
    "caching",
    "database",
]
---

There's quite a few different ways to cache with Rails, and a [lot of information about it](https://guides.rubyonrails.org/caching_with_rails.html) in the guides.

Ideally when you cache it's shared across multiple instances and across many threads. That's where Redis or Cassandra comes in, but what happens when the data you want to place in the cache store is too large?

## The problem

On [Creators](creators.brave.com) we have 400,000 users who have associated balances for their account. User's balances update regularly, and recently we've run into some hiccups with retrieving them. While optimization should happen on the Balance Server, we can add some improvements on our side to help speed things up.

One easy performance gain is to reduce the number of calls we make to the server from one every page load to once every thirty minutes. A simple way to do that is to have some caching code like this

```ruby
def balance
  Rails.cache.fetch("#{user.id}/balance", expires_in: 30.minutes) do
    UserBalanceGetter.new(user: user).perform
  end
end
```

However once we did the math we found that this was going to cause our cache store to overflow and run out of memory. If we have 100,000 users logged in we store a list of transactions and balances for each user at 5kb each that's 500 MB needed in our cache store.

If we run our monthly threaded batch processing on all of our users we're looking at upwards of 2 GB of usage. Not very scalable for an application which is growing by 1000-2000 users a day.


## Enter the database

So our solution was to introduce a simple database backed cache system.

First we created our migration for the properties.


```ruby
class AddCachingToUsers < ActiveRecord::Migration[6.0]
  def change
    add_column :users, :balance_cache, :jsonb
    add_column :users, :balance_cache_updated_at, :datetime
  end
end
```

Then we added the following properties to our models.

```ruby
class User < ApplicationRecord
  def balance_cache
    update(balance_cache: BalanceGetter.new(user: user)) if balance_cache_expired?

    read_attribute(:balance_cache)
  end

  def balance_cache=(value)
    super(value)
    write_attribute(:balance_cache_updated_at, current_time_from_proper_timezone)
  end

  private

  def balance_cache_expired?
    balance_cache_updated_at.blank? || 30.minutes.ago > balance_cache_updated_at
  end
end
```

If the cache hasn't expired we read the attribute from the model with the `read_attribute` Rails method.

If the cache has expired, or the updated time is blank we update it. This is where the setter comes in, when we set the `balance_cache_updated_at` time to be the [`current_time_from_property_timezone`](https://github.com/rails/rails/blob/dcaa388badf49423fb8613bc02652e448890f879/activerecord/lib/active_record/timestamp.rb#L67). This is implementend in Rails internal `ActiveRecord::Timestamp` module.


Finally here's [the pull request](https://github.com/brave-intl/publishers/pull/2386) where we added it to our application.
