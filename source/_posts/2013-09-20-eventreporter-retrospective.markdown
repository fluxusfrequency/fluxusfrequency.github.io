---
layout: post
title: "EventReporter retrospective"
date: 2013-09-20 09:03
---

The first couple weeks of gSchool basically consisted of: our teachers (Jeff, Katrina, Franklin, and Jorge) throwing us into the deep end and watching to see if we would sink or swim. Within the first seven days of class, we were assigned the following tutorials: Ruby in 100 Minutes, EventManager, MicroBlogger, Encryptor, [TDD With Minitest and EventManager](http://tutorials.jumpstartlab.com/academy/workshops/testing_event_manager.html), and [EnumsExercises](https://github.com/JumpstartLab/enums-exercises.git).

Then they dropped the real bomb.

[EventReporter](http://tutorials.jumpstartlab.com/projects/event_reporter.html). In this project, you "build an interactive query and reporting tool" that inspects CSV files. After loading a file, it lets the user search through the data to find entries by attributes like name, phone number, and address. [My version is here](git@github.com:fluxusfrequency/event_reporter.git).

This was the first project in which we had to engineer solutions to a problem without having our hands held every step of the way. The tutorial gives some guidance, in the form of various "paths" with guidelines to be met. I guess this is the "we do" step of the guided release process ("I do", "we do", "you do"). At any rate, we found ourselves out in the wild, trying to hack out a way to victory. Each of us had to find our own process; develop a workflow to find a way out.

I was paired with Persa for this project. The instructions said that we were each to turn in our own solution, relying on each other for support as we went. I was determined to figure out how to create a working solution, and my learning style is very independent. It seems that Persa is the same way. We didn't spend much time in collaboration. We troubleshot each others' problems, but only minimally. Mostly we each worked alone, with a little consultation from the teachers or other students.

For me, the project was basically all-consuming from Tuesday morning through Thursday morning. I'm not sure how many hours I spent on it in total, but I will say that I spent all of my time at Galvanize (about 8:00-5:00) on it each day (when I wasn't in class), plus about three hours on the bus each day, and a few extra hours at home on top of all that.

My strategy was to write tests for each of the five paths from the tutorial. Here's the code from my "Happy Path" test:

```ruby happy_path_test.rb
require 'pry'
require 'minitest'
require 'minitest/autorun'
require 'minitest/pride'
require_relative '../lib/event_reporter.rb'

class HappyPathTest < MiniTest::Test
  attr_accessor :reporter

  def setup
    @reporter = EventReporter.new
    reporter.process_and_execute("load")
  end

  def test_responds_to_load_filename
    assert_equal 'Successfully loaded event_attendees.csv.', reporter.process_and_execute("load event_attendees.csv")
  end

  def test_responds_to_load_filename_nil_with_a_default_file
    assert_equal 'Successfully loaded event_attendees.csv.', reporter.process_and_execute("load")
  end

  def test_queue_count_defaults_to_zero
    assert_equal 0, reporter.process_and_execute("queue count")
  end

  def test_responds_to_find_by_first_name
    skip
    reporter.process_and_execute("find first_name John")
    assert_equal 63, reporter.process_and_execute("queue count")
  end

  def test_responds_to_queue_clear
    skip
    reporter.process_and_execute("queue clear")
    assert_equal 0, reporter.process_and_execute("queue count")
  end

  # Some code omitted for brevity...

end

```

After writing the tests, I wrote code to make the tests pass, one at a time. I repeated this process for all five paths. By the time I was done, I had a 95% functional program. I did a few tweaks here and there, and had something that worked by Thursday morning. I did it, but it was a haul. I enjoyed reading this 10:00 PM email from Jeff on Wednesday night:

```plain Jeff's Message
All,

I imagine there are many of you who have been cranking on EventManager and there's likely a cadre still at Galvanize. Please stop where you are and get some sleep. We have good plans for tomorrow and don't want to lose a day to you being totally drained.

It will be ok :)
```

Ha ha. Right, Jeff. *That's* what we're going to do.

Meanwhile, class was still going on each day. Franklin did a great couple of sessions on debugging, and introduced us to [Pry](http://pryrepl.org/). Wow, it's amazing! I loved being able to stop code as it was running ro check out the value of different variables, and explore what would happen when calling different methods during runtime. It blew my mind, and *really* helped with debugging. Katrina opened my eyes to a new tool too: minitest/pride. In her words, "pride is very important!" I'm very excited by colors (as you can read about in [yesterday's post](blog/2013/09/19/fun-with-terminal/)), and I seriously don't know if I could have run the tests over and over without the rainbows to brighten my day (or night).

Even though the project is "done", I'm still working on it. I'm not surprised, and I'm sure this will always be the case with my code. Most of the work I'm doing now is based on the refactoring suggestions Franklin made during the code review I went through with Rolen, Will, Simon, Quentin, Ben H., and Nathaniel on Thursday morning.

Things I'm trying do to my code:

* Write method names clearly
* Shorten methods
* Split if/else statements into separate methods
* Change variable assignments into methods that return an array or hash, when appropriate
* Routing all the "puts" calls through a "say" method, so that I can run tests without seeing a bunch of garbage on the screen.

After seeing my classmates' code, and listening to Franklin's ideas, my whole understanding of Ruby syntax and ways of approaching problems has expanded many times over. It's going to take a long time to integrate everything I've seen this week into my coding.

A couple of things I did that worked really well for me were:

* Using pomodoros to keep from getting burned out
* Driving feature implementation using tests
* Splitting collections of methods into separate classes
* Using namespacing
* Using git branches and pushing often
* Drinking coffee listening to music on big headphones while coding :)

Things that didn't work as well:

* Using really big methods,
* Using a lot of puts statements at the end of my methods (making it hard to test)
* Continually switching between open windows instead of splitting them in a way that makes sense

All in all, I had a great time with EventReporter. It was fun for me to define a large number of problems, and solve them one at a time until I'd built something cool.

[Here's another link](git@github.com:fluxusfrequency/event_reporter.git) to my current version.
