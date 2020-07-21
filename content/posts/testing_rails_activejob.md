---
title: "Testing ActiveJob with Rails 6"
date: 2020-07-21
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

It can be a struggle to tests Rails sometimes. Documentation surrounding the Rails ActiveJob module isn't the best. This post will walk you through some snippets and how you can test.

I primarily use Minitest as it was the testing framework set up in the project I work in but all code that works in MiniTest should also work for RSpec.

## Asserting that something was enqueued in a Rails Test

```ruby
test "sends one email and refreshes auth token" do
  assert_enqueued_jobs(1) do
    # Sends off an email
    MailerServices::VerifyEmailEmailer.new(publisher: publisher).execute
  end
end
```

## Assert an email is enqueued

Another option is to check to see if an email is sent. You can view all the Test Helpers with [`ActionMailer::TestHelper`](https://api.rubyonrails.org/classes/ActionMailer/TestHelper.html)

```ruby
def test_emails
  assert_emails 0
  ContactMailer.welcome.deliver_now
  assert_emails 1
  ContactMailer.welcome.deliver_now
  assert_emails 2
end
```

Alternatively you can check that a specific email was enqueued

```ruby
def test_email_in_block
  assert_enqueued_email_with ContactMailer, :welcome do
    ContactMailer.welcome.deliver_later
  end
end
```

## Test a specific job was enqueued

If you have a complex application which is enqueuing a lot of jobs then you'll want to make sure the job your testing was enqueued

```ruby
test "create launches EnqueueUsersForPayoutJob for administrators" do
  sign_in users(:admin)

  assert_enqueued_with(job: EnqueueUsersForPayoutJob) do
    post admin_payout_reports_path
  end
end
```

You can even pass in arguments, so that you can ensure that something is valid

```ruby
# Pull channel from fixtures
channel = channels(:unverified)

assert_enqueued_with(job: VerifySiteChannel, args: [{ channel_id: channel.id }]) do
  # Execute job that enqueued VerifySiteChannel.
  EnqueueSiteChannelVerifications.perform_now
end
```

## Clear the existing jobs or queue

Sometimes you'll have a bunch of items in your ActiveJob queue but you only want to check that a job was enqueued after your code.

```ruby
test "approves channels that have waited the timeout period" do
  clear_enqueued_jobs
  clear_performed_jobs

  travel(Channel::CONTEST_TIMEOUT + 1.minute) do
    assert_enqueued_jobs 0
    Channels::TransferChannelsJob.perform_now

    assert_enqueued_jobs 1
  end
end
```

### Asserting how many jobs were performed

In the above example I'm asserting that jobs were enqueued, but what about jobs that were performed?

Consider the following code:

```ruby
assert_performed_jobs 0
perform_enqueued_jobs(only: Promo::RegisterChannelForPromoJob)
assert_performed_jobs Promo::AssignPromoToChannelService::MAX_ATTEMPTS
```

We assert that there was 0 performed jobs. Then we execute only the job that we want to perform. Finally we assert that our retry logic in our job was valid by asserting the performed_jobs was equal to `MAX_ATTEMPTS`.

## Asserting or checking the performed jobs

One thing that's helpful is checking the performed job was enqueued with the arguments you expected

```ruby
assert_performed_with(job: RegisterUserWithSendGridJob) do
  patch(complete_signup_users_path, params: COMPLETE_SIGNUP_PARAMS)
end
```
