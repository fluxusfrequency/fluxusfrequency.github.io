---
layout: post
title: "Getting Started With Active Job"
date: 2014-10-15 06:40:09 -0600
comments: true
categories:
---

This post was a featured article in [Ruby Weekly](http://rubyweekly.com/issues/215).

It originally appeared on [Engine Yard](https://blog.engineyard.com/2014/getting-started-with-active-job) and was also published on the [Quick Left Blog](https://quickleft.com/blog/getting-started-with-active-job/).

## Introduction

With the [announcement of Rails 4.2](http://edgeguides.rubyonrails.org/4_2_release_notes.html) came some exciting news: Rails now has built-in support for executing jobs in the background using [Active Job](https://github.com/rails/rails/tree/master/activejob). The ability to schedule newsletters, follow-up emails, and database housekeeping tasks is vital to almost any production application. In the past, developers had to hand-roll this functionality, and configuration varied between different queueing services. With the release of Rails 4.2, setting up jobs to be executed by workers at a later time is standardized. In this article, we'll take a look at how to set up Active Job and use it to send a follow-up email to a new user.

## Updating Rails

You'll need Rails 4.2.0beta1 or greater if you want to Active Job available by default (in older versions of Rails, you can require it as a gem). This tutorial is based on Rails 4.2.0beta2 (edge Rails). If you want to use edge Rails, use `gem 'rails', github: 'rails/rails'` in your `Gemfile`, and run `bundle update`.

## Setting Up Resque

In order to send emails outside of our main application process, we'll need to make use of a queueing system. There are many choices of technology for setting up background workers available, and Active Job abstracts the differences between them. Today we'll use Resque, as it's widely-used and stable.

To use Resque, you'll need to make sure you have [Redis](http://redis.io/) installed. If you don't, I recommend getting it with [Homebrew](http://brew.sh/). Otherwise, you can follow the [instructions for download](http://redis.io/download) from the official site. Once it's set up, make sure `redis-server` is running.

The next step is to install and configure the Resque gem. We'll also need `resque-scheduler` to use ActiveJob. Add them the `Gemfile` with `gem 'resque'` and `gem 'resque-scheduler'` and `bundle install`. We'll also need to create a Resque configuration file:

```ruby
#config/initializers/resque.rb

Resque.redis = Redis.new(:url => 'http://localhost:6379')
Resque.after_fork = Proc.new { ActiveRecord::Base.establish_connection }
```

We'll also require the Resque and Resque Scheduler rake tasks, so we can start our workers and scheduler with rake:

```ruby
#lib/tasks/resque.rake

require 'resque/tasks'
require 'resque/scheduler/tasks'

namespace :resque do
  task :setup do
    require 'resque'
    require 'resque-scheduler'
  end
end
```

We can now start a worker with `QUEUE=* rake environment resque:work`. If everything's working right, we should be able to see it in the Resque console. Run `resque-web` and visit `http://0.0.0.0:5678/overview`. If you see "0 of 1 Workers Working", all's well. We'll also need to boot up the scheduler in a separate process with `rake environment resque:scheduler`.

## Creating the Mailer

Now that we have a worker, we need it an email to send. Let's imagine that we want to send a follow up email to a user that recently registered for our site. We'll create a `UserMailer`, with a `follow_up_email` method that takes an email address.

```ruby
#app/mailers/user_mailer.rb

class UserMailer < ActionMailer::Base
  default from: 'noreturn@example.com'

  def follow_up_email(email)
    mail(
      to: email,
      subject: 'We hope you are enjoying our app'
    )
  end
end
```

We'll also need to write a follow-up email template.

```ruby
#app/views/user_mailer/follow_up_email.text

Hey, we saw that you recently signed up for our app.
We hope you're enjoying it!
```

## Creating the Job

Now that we have a working mailer, we can set up Active Job. All we really need to do is configure it to use the Resque adapter.

```ruby
#config/initializers/active_job.rb

ActiveJob::Base.queue_adapter = :resque
```

Next, we'll create a job that tells the background worker to send the email. The conventions for a `job` include: giving it a `queue_as`, and defining a `perform` method.

```ruby
#app/jobs/follow_up_email_job.rb

class FollowUpEmailJob < ActiveJob::Base
  queue_as :email

  def perform(email)
    UserMailer.follow_up_email(email).deliver_now
  end
end
```

Now when a user signs up, we can have the `UsersController` enqueue the job for execution at a later time. Although you would probably delay the job a few days in a real application, we'll just wait 10 seconds for easier testing.

```ruby
#app/controllers/users_controller.rb

class UsersController < ApplicationController
  def new
    @user = User.new
  end

  def create
    @user = User.create(user_params)
    FollowUpEmailJob.new(@user.email).enqueue(wait: 10.seconds)
    # redirect somewhere
  end
end
```

To make this work, we'll need some routes and a view template:

```ruby
#config/routes.rb

Rails.application.routes.draw do
  resources :users, only: [:new, :create]
end
```


```erb
#app/views/users/new.html.erb

<%= form_for @user do |f| %>
  <%= f.email_field :email %>
  <%= f.submit %>
<% end %>
```

## Setting Up Mailcatcher

Before we try our job, we'll want to make sure we can intercept the emails we're expecting the mailer to send. To achieve this, we'll use the `mailcatcher` gem. Do a global install with `gem install mailcatcher`, and run `mailcatcher`. Once we configure Action Mailer to send the emails to `localhost:1025` via smtp, we'll be able to view intercepted emails at `http://127.0.0.1:1080`.

```ruby
#config/environments/development.rb

Rails.application.configure do
  ...
  config.action_mailer.delivery_method = :smtp
  config.action_mailer.smtp_settings = { :address => "localhost", :port => 1025 }
end
```

## Trying It Out

Now everything is set up. To try it out, we'll sign up as a new user and watch the job get enqueued in the queue, then catch the mail in Mailcatcher. At this point, we have four processes running:

- `mailcatcher`
- `QUEUE=* rake environment resque:work`
- `rake environment resque:scheduler`
- `rails server`

In your browser, view the Resque dashboard at `http:/http://0.0.0.0:5678`. In another tab, visit `http://127.0.0.1:1080` to see the Mailcatcher dashboard.

Now, for the moment of truth. Visit `localhost:3000/users/new` and sign up as a new user. Ten seconds later, a new job will appear in the `emails` queue of the Resque dashboard. Just afterward, the email will appear in Mailcatcher.

## Using Acive Job with Action Mailer

The pattern we've written here woks for scheduling any job. But, since ActiveJob is now baked into ActionMailer, we can also schedule the job directly with the `UserMailer` in the `UsersController`.

```ruby
#app/controllers/users_controller.rb

class UsersController < ApplicationController
  ...
  def create
    ...
    UserMailer.follow_up_email(email).deliver_later!(wait: 10.seconds)
  end
end
```

## Conclusion

Active Job makes scheduling background jobs easier. It's also a great way to set up your job infrastructure without knowing too much about what queueing system you're using. If you needed to switch to [Sidekiq](https://github.com/mperham/sidekiq) or [Delayed Job](https://github.com/collectiveidea/delayed_job) in the future, it would be as simple as setting ActiveJob to the appropriate adapter.

It can be a little bit tricky to get Active Job set up correctly. Hopefully, this tutorial made the process a little more transparent for you.

Until next time,

Happy hacking!

