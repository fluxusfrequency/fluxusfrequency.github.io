---
layout: post
title: "Seven Reasons I Love Minitest"
date: 2014-12-18 05:36:48 -0700
comments: true
categories:
---

This post was a featured article in [Ruby Weekly](http://rubyweekly.com/issues/225).

It originally appeared on [Engine Yard](https://blog.engineyard.com/2014/seven-reasons-i-love-minitest), and was also published on the [Quick Left Blog](https://quickleft.com/blog/seven-reasons-i-love-minitest/).

## Introduction

The other day at our company standup, I mentioned that I was eager to read an article on [Concurrency in Minitest] (http://chriskottom.com/blog/2014/10/exploring-minitest-concurrency/) that was featured in [Ruby Weekly](http://rubyweekly.com/issues/216). One of my coworkers asked: "people still use Minitest?" My reply: "you mean you're not using Minitest yet?"

I love Minitest. It's small, lightweight, and ships with Ruby. It's used by respected programmers like [Aaron Patterson](http://rubyrogues.com/001-rr-testing-practices-and-tools/), Katrina Owen, Sandi Metz, and of course, DHH. Here's a look at why Minitest remains a powerful and popular choice for testing Ruby code.

## Witness the Firepower of this Fully Armed and Operational Testing Tool

Although I come from a [family of programmers](http://fluxusfrequency.github.io/blog/2013/10/03/the-history-of-programming-in-the-life-of-russ-lewis/), I entered the profession by going to a bootcamp.  I was a student in the second [gSchool](http://www.galvanize.it/school/) class. At that time, it was being taught by [Jumpstart Lab](http://jumpstartlab.com/).  My instructors were [Jeff Casimir](https://twitter.com/j3), [Franklin Webber](https://twitter.com/franklinwebber), and [Katrina Owen] (https://twitter.com/kytrinyx).

My classmates and I were brought up with TDD from day one. We practiced it in everything we did. The tool we used was Minitest. When it was first introduced, I scoffed a little because of the name.

"Why are we using a 'mini' testing framework?  I want to use what the pros use,"
I complained to Katrina.

"Minitest is a fully-featured testing framework, and is used in plenty of production apps," was her reply.

I've been using it ever since. Here are seven reasons I think Minitest is the bees' knees.

## 1. It's Simple

Minitest ships with Ruby because it's easy to understand, and can be written by anyone who knows the language.  I love this simplicity because it makes it easy to focus on designing code.  Just imagine what you want it to do, and write your assertion, and make it pass.

I recently reached out to Katrina to ask her thoughts on what's good about Minitest. Its simplicity was at the top of her list:

"It's simpler. There's no 'magic' (just plain Ruby) [...] When there is "magic" then it's very easy to assume that you *can't* understand it [...] You can read through the [Minitest] source code, and understand what is going on."

I also asked Sandi Metz what she thought of Minitest. She said:

"Minitest is wonderfully simple and encourages this same simplicity in tests and code."

Minitest is low in sugar. All you need to know is `assert` for booleans, and `assert_equal` for everything else. If you want to test a negative case, use `refute` or `refute_equal` instead. For everything else, just write Ruby.

## 2. It's Extensible

The relatively "low-level" status of Minitest makes it easy to customize tests. Katrina observes:

"Because Minitest is so simple, it's not too hard to extend. It feels like a tool that I can shape to my needs, rather than a tool that I need to use the way it was intended."

If you want to repeat an assertion in multiple contexts, you can write a custom assertion for it, and call it in as many tests as you need.

```ruby
def assert_average_speed(swallow)
  # Do some additional work here
  assert_equal '11 MPS', swallow.speed
end

def test_african
  swallow = Swallow.new(type: 'African')
  assert_average_speed(swallow)
end

def test_european
  swallow = Swallow.new(type: 'European')
  assert_average_speed(swallow)
end
```

If you are testing something that requires nearly the same test to be run repeatedly, for example when testing a controller that has a user authentication `before_action`, you can create a shared example by [including a module](https://canaryup.com/blog/shared-examples-with-minitest).

If you need even more control, you can create an object with custom behavior that inherits from `Minitest::Test`, and have your tests inherit from it. Doing so allows you to completely customize your test suite with new methods as you see fit.

Finally, Minitest comes with hooks to help you easily write extensions to the gem. You can define a plugin by adding a `minitest/XXX_plugin.rb` file to your project folder, and Minitest will automatically find and require it. You can use extensions to define command-line options, custom reporters, and anything else you want Minitest to be able to do.

## 3. It's Flat If You Use Assert

The default syntax for Minitest is `assert`. Although the BDD `expect` syntax is available in [mintest/spec](https://github.com/seattlerb/minitest/blob/master/lib/minitest/spec.rb), I recommend giving `assert` a try. Maybe you love expectations because they read like English, sort of. Although `assert` doesn't try to imitate natural language, it's actually quite intuitive to use. It also provides the benefit of making your test files flat.

With nested `context`, `describe`, and `it` blocks, it can be difficult to remember what `it` refers to, and which `before` blocks are accessible in your scope. I find myself scanning indentation in BDD tests to figure out what scope I'm working in.

When you use `assert` syntax, your test file is flat. You start with a test class, then you write a bunch of test methods inside of it. It's clear that the only variables available are those defined in the `setup` method or in the test itself. This flatness also means it gets painful quickly if your test is tied to too many dependencies. As Katrina puts it, "the lack of nested contexts means that I'm faced with the appropriate amount of pain when I make bad choices. It quickly becomes ugly if I have too many dependencies. I like that."

Using `assert` also makes it easy to document the desired behavior of your code: just name the test method to describe what you are testing. If you're really worried you'll forget what the test was for, you can output a message to the console if the test fails:

```ruby
test 'it calculates the air-speed velocity of an unladen swallow' do
  swallow = Swallow.new(wing_span: 30, laden: false, type: 'European')
  expected = '11 MPS'
  actual = swallow.average_speed
  assert_equal expected, actual, 'The average speed of an unladen
    swallow was incorrect'
end
```

Minitest's flatness is also beneficial when it comes to practicing a Test-Driven workflow. You can `skip` all of the tests in a file easily, without scanning through nested blocks. Then you can make them pass, one at a time.

## 4. It Lends Itself to A Good Test-Driven Workflow

Minitest is awesome for getting into a red/green/refactor loop. Write a test, watch it fail, make it pass, refactor. Repeat. A Minitest file is just a list of tests that are waiting for you to make them pass.

Plus, since you're just writing Ruby, you can use a style like this to get the next test set up with a minimum of effort:

```ruby
test 'something' do
  expected = # some code
  actual = # some simple result
  assert_equal expected, actual
end
```

If you want to repeat an assertion in different contexts, you can write a method for it, and call it in as many tests as you want to. Need a [shared example](https://canaryup.com/blog/shared-examples-with-minitest)? Include a module.

## 5. Minitest::Benchmark is Awesome

If you are dealing with large amounts of data, and performance is a concern, [Minitest::Benchmark](https://github.com/seattlerb/minitest/blob/master/lib/minitest/benchmark.rb) is your new best friend.

It lets you test your algorithms in a repeatable manner, to make sure that their algorithmic efficiency don't accidentally get changed. You can collect benchmarks in "tab-separated format, making it easy to paste into a spreadsheet for graphing or further analysis".

Here are a few of the assertions in Minitest::Benchmark that might be of interest:

- `assert_performance_constant`
- `assert_performance_exponential`
- `assert_performance_logarithmic`
- `assert_performance_linear`

## 6. It's Randomized By Default

Running the tests in a different order each time can help catch bugs caused by unintended dependencies between examples. Here's how Aaron Patterson described the benefit of randomized tests in an [episode of Ruby Rogues](http://rubyrogues.com/001-rr-testing-practices-and-tools/):

"I’m sure you’ve been in a situation where your [...] test fails in isolation, but when you run [the entire test suite] it works. [T]hat’s typically because one test set up some particular environment that another test depended on. [...] Since Minitest runs the test in a random order, you can’t make one test depend another, so you’ll see an error case."

## 7. It's Faster

Minitest doesn't create any matchers or example objects that have to be garbage collected. Many people have benchmarked Minitest against other frameworks, and it usually comes out at ahead. Often it's a marginal difference, but sometimes it comes out [significantly ahead](http://blog.rawonrails.com/2012/01/very-cursory-test-of-rspec-28-speed.html).

Minitest also supports concurrent test runs. Although I thought this would lead to great speed gains, it turns out that it [only makes a difference in JRuby and Rubinius](http://chriskottom.com/blog/2014/10/exploring-minitest-concurrency/). The Matz Ruby Implementation (MRI) [doesn't get speed gains](https://blog.engineyard.com/2010/concurrency-real-and-imagined-in-mri-threads) when using concurrency. Still, it's nice to know that the option is there, in case you are using JRuby or Rubinius, or the MRI changes in the future.

## Have You Tried It?

I've talked to a lot of people that are surprised when I say I prefer Minitest. They often ask, "why should I switch to Minitest?" Perhaps a better question to ask is this one, posed by [Ken Collins](https://github.com/metaskills/holy_grail_harness/issues/3): "What is in [other testing frameworks] that you need that Minitest does not offer?"

Programming tools are a matter of individual taste. When I've asked programmers about Minitest, they've all expressed that they were not _against_ RSpec or other frameworks. Sandi Metz said: "I'm agnostic about testing frameworks [...] I also use RSpec and I find it equally useful in it's own way." Katrina Owen said: "I don't dislike RSpec, but I do prefer Minitest." In the end, I think most would agree that testing frameworks are a personal choice.

That said, if you haven't used Minitest lately (or ever), why not check it out?  If you're curious, but feel like you still need some convincing, just try it!  It's a small investment. You'll be up and running in a few minutes. Why not go try the first Ruby [Exercism](http://exercism.io/getting-started)?

I hope this tour of Minitest's features has been informative, and piqued your interest in this fabulous test framework. Is there anything I missed? Disagree? Let's talk! Please share your point of view in the comments section, or tweet at me at @fluxusfrequency.

## Other Resources

[A Big Look At Minitest](http://www.slideshare.net/markykang/minitest-2013-0910)
<br />
An informative slide deck that explores the ins and outs of Minitest.

[Bow Before Minitest](https://speakerdeck.com/ahawkins/bow-before-minitest)
<br />
A more opinionated slide deck comparing Minitest with other testing
tools.

