---
layout: post
title: "Five Capybara Hacks To Make Your Testing Experience Less Painful"
date: 2014-05-20 06:20:08 -0600
---

This post originally appeared on the [Quick Left Blog](http://quickleft.com/blog/five-capybara-hacks-to-make-your-testing-experience-less-painful).

## Testing, Testing...

Everyone knows it's important to test your code. But sometimes, the experience
can be a little bit painful. Ok, sometimes it's very painful.

![Painful Testing](http://quickleft-production.s3.amazonaws.com/uploads/asset/attachment/58/asset.jpg)

What to do? Abandon the tests? Never! Smokey, this is not 'Nam. This is coding. There are rules.

Today I want to share a few of the things I've learned that help mitigate that
pain when testing Rails with Capybara. Ladies and gentlemen, I give you...

## The Hacks

### 1) Execute Script
In one project I was working on, I wanted to use a fake password input field so
I could display a text field instead of a password field. I wanted to do
that so I could give it placeholder text that wouldn't appear as dots.
When the fake field received focus, it was replaced with the real password
field using jQuery.

This solution passed the click test, but it was a headache when testing.
My Capybara spec couldn't find the password field, because it was hidden.
Since pretty much every integration test I had needed to log in before it could
do anything, I was stuck.

In order to overcome the difficulty, I had the test execute a script to show the
field I was looking for.

```ruby
  page.execute_script("$('#new_password').show()")
```

I could then fill out the form normally and access the rest of the app.

This trick also comes in useful when using Bootstrap for responsive design.
Sometimes, the mobile menu dropdown can cause conflicts with other elements on
the page, and Capybara can't click on them.

### 2) Tail the Test Log File

I've gotten so used to being able to read the Rails server stack trace when
troubleshooting in development, that I sometimes I don't know what to do to when
I want to find my errors in while testing. The test stack trace and pry will only
take you so far. When things get really hairy, you can get a similar output
as you would see in development by running `tail -f log/test.log` in a separate
terminal tab before running the specs.

### 3) Find the Port Address and Pause the Test

Sometimes I just want to see what's going on in the browser. Although
`save_and_open_page` gives you some idea of what's going on, you can't
really click around, because the page it gives you is static and lacking assets.
To dig into what's going on, I like to use a trick I learned from my mentor,
Mike Pack.

Just above the broken line in your test, add these two lines of code.
Note that you have to have pry installed in your current gemset or
specified in the Gemfile for this to work.

```ruby
puts current_url
require 'pry'; binding.pry
```

Run the specs, and when they pause, copy the url and port number from the test
output. Open your browser and paste the address into the window. Voila!
You're now browsing your site in test mode!

### 4) Make Sure DatabaseCleaner Plays Nice with PhantomJS

It can get really tricky to test JavaScript in Rails.
Arguably, using some Jasmine tests may be the best thing if you have
a lot of JS code. That may work for unit tests, but if you want to test
a feature from end to end, it usually makes sense to use Capybara. To drive the
JavaScript, you need an engine like [Poltergeist](https://github.com/teampoltergeist/poltergeist), which relies on [PhantomJS](http://phantomjs.org/).

Meanwhile, you also need to take care of cleaning the database between specs to
make sure that they are independent. In my apps, I usually take care of this
with the [DatabaseCleaner](https://github.com/bmabey/database_cleaner) gem.

Unfortunately, PhantomJS runs the JavaScript code in a separate thread from the
application code, which means that JS transactions don't always get
committed to the database before DatabaseCleaner attempts to clean them.

The answer is to make sure that you set it to use the `:truncation`
strategy for JavaScript tests. See Avdi Grimm's
[blog post](http://devblog.avdi.org/2012/08/31/configuring-database_cleaner-with-rails-rspec-capybara-and-selenium/) on the subject for more details.

Here's how to set it up in your `spec_helper`:

```ruby
RSpec.configure do |config|

  config.before :suite do
    DatabaseCleaner.strategy = :truncation
  end

  config.before(:each) do
    DatabaseCleaner.strategy = :transaction
  end

  config.before(:each, :js => true) do
    DatabaseCleaner.strategy = :truncation
  end
end
```

### 5) Split VCR/Webmock Specs Into a Separate Rake Task From Your JavaScript Tests

Sometimes I have service objects that interact with an external API. The easiest
way to test them without relying on actual HTTP requests is to use the
[VCR](https://github.com/vcr/vcr) gem. With VCR, you can also hook into
[Webmock](https://github.com/bblimke/webmock), which keeps your computer from
making any external HTTP requests during the test cycle. This is all well and good
until you are making requests against a local API with JavaScript as well.
In that case, Webmock will block the requests. Over the line!

I've played around with various ways of setting a condition in my test
helper and before blocks to only run Webmock for a certain test, but I've come
up with another solution that I prefer. Instead of running `rspec`, I split the
tests into a `:services` group, and a `:local` group. Then, I write a rake task
in lib/tasks called `spec.rake` that looks like this:

```ruby
namespace :spec do
  desc "run all local specs"
  task :local  do
    system 'rspec spec/models'
    system 'rspec spec/features'
    system 'rspec spec/controllers'
  end

  desc "run all service specs"
  task :services do
    system 'rspec spec/services'
  end

  task :all do
    system 'rspec spec/models'
    system 'rspec spec/features'
    system 'rspec spec/controllers'
    system 'rspec spec/services'
  end
end
```

This way, you only have to type one command to run all of your specs, just as
you would if you had typed `rspec`. It's also easy to hook it into your Travis CI
configuration like this:

```ruby
language: ruby
rvm:
  - "2.0.0-p353"
script:
  - bundle exec rake db:create
  - bundle exec rake db:migrate RAILS_ENV=test
  - bundle exec rake spec:local
  - bundle exec rake spec:services
```

Boom! Local and external APIs all tested, JavaScript tested, and Travis
build passing.

## Conclusion

If you're using a lot of JavaScript (and who isn't these days?) in your Rails app, it can sometimes hurt to write your feature tests. I hope these five hacks will make your life a little easier when you are testing, so you can get back to writing the code that makes your app go. Until then, 'the Dude abides'.

![The Dude Abides](http://quickleft-production.s3.amazonaws.com/uploads/asset/attachment/59/asset.gif)
