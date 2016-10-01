---
title: Learning TDD
date: 2013-09-17 06:01:00 Z
categories:
- source
layout: post
---

We're now into the second week of gSchool. It has been an intense six days thus far. The Jumpstart Lab crew has really thrown us into the deep end, expecting us to have completed five tutorials already. One more, [EventReporter](http://tutorials.jumpstartlab.com/projects/event_reporter.html), is due at the beginning of day nine.

What we've already done:

- [Ruby In 100 Minutes](http://tutorials.jumpstartlab.com/projects/ruby_in_100_minutes.html)
- [EventManager](http://tutorials.jumpstartlab.com/projects/eventmanager.html)
- [Encryptor](http://tutorials.jumpstartlab.com/projects/encryptor.html)
- [MicroBlogger](http://tutorials.jumpstartlab.com/projects/microblogger.html)
- [TDD With MiniTest And EventManager](http://tutorials.jumpstartlab.com/academy/workshops/testing_event_manager.html)

This last tutorial was yesterday's project. I was really excited to get into writing some tests. We talked about testing in class, and the consensus was that Test Driven Development did lots of good things for programmers. Some of the students' explanations of testing included: it helps make sure the application is working, it helps guide the developer to the next features that need to be implemented, it describes the application. I am excited to start including it in my workflow, because all of the error-driven clicking and typing I've been doing up until now is getting really old.

Before bootcamp started, I worked through all of these tutorials on my own, and wrote [my own copy](https://github.com/fluxusfrequency/treebook.git) of the Treebook tutorial on [Treehouse](http://www.teamtreehouse.com) as well. While completing Treebook, Jason Siefer walks you through a lot of TDD, first using MiniTest, and later Rspec and the Shoulda gem. Now that I'm digging into testing without the support of a tutorial, I'm trying to decide which syntax I like best for testing. Although it is the simplest, I'm not a huge fan of code like this:

```ruby EventReporter MiniTest
require 'minitest'
require 'minitest/autorun'
require_relative '../lib/event_reporter.rb'

class HappyPathTest < MiniTest::Test
  attr_accessor :reporter

  def setup
    @reporter = EventReporter.new
  end

  def test_queue_count_defaults_to_zero
    assert_equal 0, reporter.queue_count
  end

  def test_responds_to_find_by_first_name
    reporter.find_by(:first_name, "John")
    assert_equal 63, reporter.queue_count
  end
```

The syntax is just ugly, to me. I prefer this Rspec code I wrote for Treebook:

```ruby Treebook Friendship Decorator Test
require 'test_helper'

class UserFriendshipDecoratorTest < Draper::TestCase
  context "#sub_message" do
    setup do
      @friend = create(:user, first_name: 'Fred')
    end

    context "with a pending user friendship" do
      setup do
        @user_friendship = create(:pending_user_friendship, friend: @friend)
        @decorator = UserFriendshipDecorator.decorate(@user_friendship)
      end

      should "return the correct message" do
        assert_equal "Friend request pending.", @decorator.sub_message
      end
    end
```

I noticed last night that my fellow student, Katrina Engelstad, tweeted a link to this [blog post](http://www.mattsears.com/articles/2011/12/10/minitest-quick-reference) by Matt Sears. It looks like a useful reference for MiniTest. Here's a block of code from that page which shows a way to make MiniTest behave like Rspec:

```ruby Describe Hipster http://www.mattsears.com/articles/2011/12/10/minitest-quick-reference
require 'minitest/autorun'

describe Hipster, "Demonstration of MiniTest" do
  before do
    @hipster = Hipster.new
  end

  after do
    @hipster.destroy!
  end

  subject do
    Array.new.tap do |attributes|
      attributes << "silly hats"
      attributes << "skinny jeans"
    end
  end

  let(:list) { Array.new }

  describe "when asked about the font" do
    it "should be helvetica" do
      @hipster.preferred_font.must_equal "helvetica"
    end
  end

  describe "when asked about mainstream" do
    it "won't be mainstream" do
      @hipster.mainstream?.wont_equal true
    end
  end
end
```

Finally, I've been tantalized by the possibilities of [Cucumber](http://cukes.info/). I haven't had time to dig into this testing approach yet, but the prospect of writing language like this is very appealing:

```ruby How To Test With Cucumber https://github.com/plataformatec/devise/wiki/How-To:-Test-with-Cucumber
Scenario Outline: Creating a new account
    Given I am not authenticated
    When I go to register # define this path mapping in features/support/paths.rb, usually as '/users/sign_up'
    And I fill in "user_email" with "<email>"
    And I fill in "user_password" with "<password>"
    And I fill in "user_password_confirmation" with "<password>"
    And I press "Sign up"
    Then I should see "logged in as <email>" # your work!

    Examples:
      | email           | password   |
      | testing@man.net | secretpass |
      | foo@bar.com     | fr33z3     |

Scenario: Willing to edit my account
    Given I am a new, authenticated user # beyond this step, your work!
    When I want to edit my account
    Then I should see the account initialization form
    And I should see "Your account has not been initialized yet. Do it now!"
    # And more view checking stuff
```

For now, even though I think it's ugly, I'm going to stick to MiniTest for the "TDD with MiniTest and EventManager" and EventReporter tutorials, since I am still getting comfortable with TDD. Overlearning MiniTest will be a good foundation to have, even if I end up using Rspec or Cucumber in the future. So long for now! I'm off to go implement some fifty odd tests for EventReporter.
