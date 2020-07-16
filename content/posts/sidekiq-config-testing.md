---
title: "Sidekiq Config Testing"
date: 2020-07-15T13:28:01-05:00
draft: false
categories: [
  "rails"
]
tags: [
  "ruby",
  "rails",
  "testing"
]
---

## Ensure your Sidekiq configuration is correct

Often times during a refactor there can be things missed. That's one of the reasons why we write unit tests. Unfortunately I ran into the exact situation where I'm describing, I [renamed one of job's modules](https://github.com/brave-intl/publishers/pull/2761) from `Channel` to `Channels` but I forgot to update the `sidekiq.yml` file. 🤦‍♂️

The worst part was finding this much later and only after users were complaining about the job not working.

To counteract that I wrote the following unit test for ensuring valid configuration.


```ruby
require 'test_helper'

class SidekiqConfigurationTest < ActiveJob::TestCase
  test "sidekiq.yml contains only valid classes" do
    file_path = Rails.root.join('config/sidekiq.yml')

    configuration = YAML.load_file(file_path)

    assert configuration.present?

    jobs = configuration.dig(:schedule).keys
    jobs.each do |job|
      # Rails can convert a string to a class with constantize
      # https://api.rubyonrails.org/classes/ActiveSupport/Inflector.html#method-i-constantize
      # This raises a NameError if the configuration isn't valid
      job.constantize
    end
  end
end
```

There we have it! A simple, straight to the point Sidekiq job configuration test. There's a lot more you can do with this rather than just test the class exists. We could kick off a test of each job and ensure that all of them ran successfully. We could test the queue names to ensure that each job is correctly assigned to a queue. Or we could do the opposite, check all the items in `app/jobs` to ensure there is an associated entry in the `sidekiq.yml` file.

