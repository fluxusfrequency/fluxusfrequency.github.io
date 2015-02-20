---
layout: post
title: "Wrapping Your API In A Custom Ruby Gem"
date: 2014-10-02 07:03:19 -0600
---

This post originally appeared on [Engine Yard](https://blog.engineyard.com/2014/wrapping-your-api-in-a-ruby-gem).
It also appeard on the [Quick Left Blog](https://quickleft.com/blog/wrapping-your-api-in-a-custom-ruby-gem/).

## Introduction

In the modern web, API-based projects are becoming the norm. Why? For one thing, APIs are necessary to serve Single Page Applications, which are all the rage right now. From a business standpoint, APIs give companies a new way to charge others for access to their data. If you are part of a company that offers such a service, a great way to generate interest in your API is to offer a Ruby gem that makes fetching and consuming your data easy for Ruby developers.

Today, we'll take a look at how to wrap an imaginary API in a new Ruby gem and share it with the world. If you want to follow along at home, you can [clone the project from my GitHub account](https://github.com/fluxusfrequency/benzinator).

## Our API

Let's pretend we have an application called Ben's Benzes that serves data about cars for sale. We're exposing a RESTful API so that developers from other companies can serve our car data on their websites. For our first iteration, here are the routes we've set up:

Get info about a certain car currently for sale:
`GET http://www.bensbenzes.com/api/v1/cars/active/:id`

Get all the cars that are currently for sale:
`GET http://www.bensbenzes.com/api/v1/cars/active`

## Setting Up the Gem

We'll be using [bundler](http://bundler.io/), so begin by making sure you have that installed (you probably do). We'll create the gem by running `bundle gem <gem-name>` from the command line. I'm going to use `benzinator` as the name of my gem. That name is now taken, so you'll have to come up with your own. Open up the project directory and you should see:

```
├── Gemfile
├── LICENSE.txt
├── README.md
├── Rakefile
├── benzinator.gemspec
└── lib
    ├── benzinator
    │   └── version.rb
    └── benzinator.rb
```

Let's open up `benzinator.gemspec` and do a little configuration. Update the following lines with your name, email and a summary and description of the gem.

```ruby
...
spec.authors       = ["Ben Lewis"]
spec.email         = ["blewis@example.com"]
spec.summary       = %q{Gem to wrap BensBenzes.com API}
spec.description   = %q{Gem to wrap BensBenzes.com API}
...
```

While we're in here, let's add the dependencies we'll be using, right after `bundler` and `rake`. We'll add testing tools as development dependencies, as well as hard dependencies on [Faraday](https://github.com/lostisland/faraday) and [json](http://www.ruby-doc.org/stdlib-2.1.2/libdoc/json/rdoc/JSON.html).

```ruby
...
spec.add_development_dependency "minitest"
spec.add_development_dependency "vcr"
spec.add_development_dependency "webmock"

spec.add_dependency "faraday"
spec.add_dependency "json"
...

```

## Writing a Test

We'll need to make a `test` folder, and put a `test_helper.rb` into it. We'll use the helper to pull in our gem and testing dependencies. We'll also configure [VCR](https://github.com/vcr/vcr) and [Webmock](https://github.com/bblimke/webmock), which we're using to stub out our server responses so that our gem isn't dependent on access to the API for testing. See the VCR documentation for more about how this works.

```ruby
#test/test_helper.rb
require './lib/benzinator'
require 'minitest/autorun'
require 'webmock/minitest'
require 'vcr'

VCR.configure do |c|
  c.cassette_library_dir = "test/fixtures"
  c.hook_into :webmock
end
```

Don't forget to create the `test/fixtures` folder so VCR has somewhere to put the fixtures. With that done, we'll write the first test for our gem. Our goal is to create an object called Benzinator::Car that exposes `#all` and `#find` methods to wrap calls to our API. Let's start by making sure that object exists:

```ruby
#test/car/car_test.rb
require './test/test_helper'

class BenzinatorCarTest < Minitest::Test
  def test_exists
    assert Benzinator::Car
  end
end
```

## Creating The Wrapper Model

If we now run `ruby test/car/car_test.rb`, we get this error: `uninitialized constant Benzinator::Car`. Looks like it's time to write some code:

```ruby
#lib/benzinator/car.rb
module Benzinator
  class Car
  end
end
```

We'll also need to require it in `lib/benzinator.rb`:

```ruby
require_relative "benzinator/version"
require_relative "benzinator/car"
...
```

Now if we run the test, it passes. Let's write another one to make sure that our `Benzinator::Car` model can give back the data for a car. Let's imagine that an API call to `http://www.bensbenzes.com/api/v1/cars/active/68` returns this JSON object:

```json
"{
  \"id\": 68,
  \"make\": \"Honda\",
  \"model\": \"Civic\",
  \"year\": \"1996\",
  \"color\": \"Blue\",
  \"vin\": \"XXXXXXXXXXXXXX\",
  \"dealer_id\": 34
}"
```

We'll want to make sure that our Benzinator::Car object has getter convenience methods all of the fields shown for each car.
Given these API results, we could write a test like this:

```ruby
#test/car/car_test.rb
  ...
  def test_it_gives_back_a_single_car
    VCR.use_cassette('one_car') do
      car = Benzinator::Car.find(68)
      assert_equal Benzinator::Car, car.class

      # Check that the fields are accessible by our model
      assert_equal 68, car.id
      assert_equal "Honda", car.make
      assert_equal "Civic", car.model
      assert_equal "1996", car.year
      assert_equal "Blue", car.color
      assert_equal "XXXXXXXXXXXXXX", car.vin
      assert_equal 34, car.dealer_id
    end
  end
end

```

Running this test, we get `undefined method 'find' for Benzinator::Car:Class`. Let's go define it.

```ruby
#lib/benzinator/car.rb
require 'faraday'
require 'json'

API_URL = "http://www.bensbenzes.com/api/v1/cars/active"

module Benzinator
  class Car
    def self.find(id)
      response = Faraday.get("#{API_URL}/#{id}")
      attributes = JSON.parse(response.body)
    end
  end
end
```

Now the test says we forgot to make it a Benzinator::Car model:

```
Expected: Benzinator::Car
Actual: Hash
```

We can fix that, and make the attributes into getters at the same time.

```ruby
#lib/benzinator/car.rb
...
module Benzinator
  class Car
    attr_reader :id, :make, :model, :year, :color, :vin, :dealer_id
    def initialize(attributes)
      @id = attributes["id"]
      @make = attributes["make"]
      @model = attributes["model"]
      @year = attributes["year"]
      @color = attributes["color"]
      @vin = attributes["vin"]
      @dealer_id = attributes["dealer_id"]
    end

    def self.find(id)
      ...
      new(attributes)
    end
    ...
  end
end
```

That should take care of it. Now to test the `#all` method.

Let's imagine that a call to `http://www.bensbenzes.com/api/v1/cars/active` responds with an array of 64 cars, with this Honda being the first one. We can rely on the API call giving back 64 cars today, but we hope there will be 6000 listed tomorrow. This is why we're using VCR to save the result of the call as a fixture.


```ruby
#test/car/car_test.rb
class BenzinatorCarTest < Minitest::Test
  ...
  def test_it_gives_back_all_the_cars
    VCR.use_cassette('all_cars') do
      result = Benzinator::Car.all

      # Make sure we got all the cars
      assert_equal 64, result.length

      # Make sure that the JSON was parsed
      assert result.kind_of?(Array)
      assert result.first.kind_of?(Benzinator::Car)
    end
  end
end

```

Running this, we get `undefined method 'all' for Benzinator::Car:Class`. Let's define it:

```ruby
#lib/benzinator/car.rb
...
module Benzinator
  class Car
    ...
    def self.all
      response = Faraday.get(API_URL)
      cars = JSON.parse(response.body)
      cars.map { |attributes| new(attributes) }
    end
  end
end
```

Sweet success! The tests pass!

## Publishing and Using Our Gem

Now that our gem works, we can publish it to [RubyGems](https://rubygems.org). This is a pretty easy process. First, we'll bump the version to 1.0:

```ruby
#lib/benzinator/version.rb
module Benzinator
  VERSION = "0.1.0"
end
```

Then we can bundle it up by running `gem build benzinator.gemspec`. This will create a `benzinator-0.1.0.gem` file in our project directory. To publish it , all we have to do is run `gem push benzinator-0.1.0.gem`. You'll be prompted for your RubyGems username and password, which you'll need to [create](https://rubygems.org/sign_up) if you don't have an account yet. After you enter your credentials, your gem is live!

Now anyone can use our gem in their Ruby projects. All they have to do is add it to their Gemfile with `gem 'benzinator'`, and run bundle.

## Conclusion

Winning! Now the whole wide world can access the Ben's Benzes API from their Ruby project, with convenience methods to make things easier to work with.

For our next iteration, we could get a lot more in depth. We might want to add the ability to create, edit or destroy cars. If we decided to that, we might first build sort of authentication process into the gem. Once users have gotten a taste of our data and rely on it, our API might get so popular that we need to limit the number of API calls a user can make per day. We could then write subscription service to allow users greater access to your data at a cost.

One last note: you might want to use this technique to wrap someone else's API, too! If you're using a service that doesn't offer a gem for its API, you can always write one and release it as open source!

Happy hacking!

