---
layout: post
title: "Instances, Classes, and Modules, Oh My!"
date: 2015-01-29 21:05:58 -0700
comments: true
categories:
---

This post originally appeared on [Engine Yard](https://blog.engineyard.com/2015/instances-classes-and-modules) and was later published on the [Quick Left Blog](https://quickleft.com/blog/instances-classes-and-modules-oh-my/).

## Introduction

One of the biggest challenges of object oriented programming in Ruby is defining the interface of your objects. In other languages, such as Java, there is an explicit way to define an interface that you must conform to. But in Ruby, it's up to you.

Compounding with this difficulty is the problem of deciding which object should own a method that you want to write. Trying to choose between modules, class methods, instance methods, structs, and lambdas can be overwhelming.

In this post, we'll look at several ways to solve the Exercism [Leap Year](http://exercism.io/nitpick/ruby/leap) problem, exploring different levels of method visiblitiy and scope level along the way.

## Leap Year

If you sign up for [Exercism.io](http://exercism.io/) and start solving Ruby problems, one of the first problems you will look at is called Leap Year. The tests guide you to write a solution that has an interface that works like this:

```ruby
Year.leap?(1984)
#=> true
```

A leap year is defined as a year that is divisible by four, unless it's also divisible by 100. Apparently, centuries aren't leap years. That is, unless they are centuries that are divisible by 400. So there are three rules for leap years:

1) If it's divisible by 400, it's an exceptional century and is a leap year.
2) If it's divisible by 100, it's a mundane century and in not a leap year.
3) Otherwise, if it's divisible by 4, it's a leap year, otherwise it's not.

## The First Approach: Using Class Methods

Here's one simple solution to this problem. You can rearrange the booleans in a couple of different ways and it will still work. This is the version I came up with:

```ruby
class Year
  def self.leap?(year)
    year % 4 == 0 && !(year % 100 == 0) || year % 400 == 0
  end
end
```

This is nice and compact, but not entirely easy to understand. I'm a pretty big fan of self-documenting code, so I used the [extract method](http://fluxusfrequency.github.io/blog/2014/01/10/refactoring-1-extract-method/) refactoring pattern to name these three rules.

```ruby
class Year
  def self.leap?(year)
    mundane_leap?(year) || exceptional_century?(year)
  end

  def self.mundane_leap?(year)
    year % 4 == 0 && !century?
  end

  def self.century?(year)
    year % 100 == 0
  end

  def self.exceptional_century?(year)
    year % 400 == 0
  end
end
```

## Class Methods and Privacy

This is a lot more understandable, in my mind. But there's a problem with this. `mundane_leap?`, `century?`, and `exceptional_century?` are all publicly exposed methods. They really only exist in support of `leap?`, and I'm not sure how reusable they are, with the possible exception of `century?`. If I write this test, it will pass:

```ruby
  def test_exceptional_century
    assert Year.exceptional_century?(2400)
  end
```

I would like to make `exceptional_century?` private, so that it can't be accessed outside of the `Year` class. I can try something like this:

```ruby
class Year
  ...

  private

  def self.mundane_leap?(year)
    year % 4 == 0 && !century?
  end

  def self.century?(year)
    year % 100 == 0
  end

  def self.exceptional_century?(year)
    year % 400 == 0
  end
end
```

But, unfortunately, this won't work. The test still passes, becuase the `private` keyword doesn't work on class methods. Instead, I would have to use `private_class_method` after my method definition.

```ruby
  private_class_method :mundane_leap?, :century?, :exceptional_century?
```

Now if I run that last test, it will raise an error.

## All About the Eigenclass

In my mind, what we've now done is somewhat better, but there's still a smell here. I'll get to exactly what that is in just a moment, but for now I'll say that it's due to the fact that we've defined all of these methods on the singleton class, or eigenclass, of `Year`. If you don't know about eigenclasses, you can read about them [here](http://madebydna.com/all/code/2011/06/24/eigenclasses-demystified.html).

We can define methods on an eigenclass by prepending each method definition with `self.`, or we can use `class << self` or `class << Year` and nesting our method definitions inside of that block. Doing it this way makes it possible to use the `private` keyword, because we're now working at the instance level of scope. If we were to introduce `class << self`, then, we could do away with our `private_class_method` call.

```ruby
class Year
  class << self
    def leap?(year)
      mundane_leap?(year) || exceptional_century?(year)
    end

    private

    def mundane_leap?(year)
      year % 4 == 0 && !century?(year)
    end

    def century?(year)
      year % 100 == 0
    end

    def exceptional_century?(year)
      year % 400 == 0
    end
  end
end
```

But we haven't really changed much here. In my mind, there's still a big smell, which is this. [According to Wikipedia](http://en.wikipedia.org/wiki/Class_%28computer_programming%29), a class is (emphasis mine) " an extensible program-code-template for _creating objects_ ". Our `Year` class never creates a single instance. It's just an eigenclass that happens to be able to create (mostly) useless `Year` instances.

So where _should_ we be putting class-level methods that are not associated with any instance object?

## Second Approach: Module Functions

In Ruby, the `Class` object inherits from `Module`. A module is basically a collection of methods, constants, and classes. Their primary feature is that they can be mixed into other modules and classes to extend their functionality. They're also frequently used for namespacing (for example, the `ActiveRecord` constant is a module).

We can put our `Year` implementation into a module without changing much: just swap the word `class` for `module`. Instead of using `class << self` as we did for the class version, we can use `extend self` in our module, and get the same effect, allowing us to use the `private` keyword. `extend` takes the methods defined in a module and makes them into class-level methods on the target module (or class). This is in contrast with `include`, which mixes them in as instance-level methods. Thus, if we extend the module into itself, it gets all of it's own methods at the class level.

```ruby
module Year
  extend self

  def leap?(year)
    mundane_leap?(year) || exceptional_century?(year)
  end

  private

  def mundane_leap?(year)
    year % 4 == 0 && !century?(year)
  end

  def century?(year)
    year % 100 == 0
  end

  def exceptional_century?(year)
    year % 400 == 0
  end
end
```

If we wanted to define some other methods in a `Year` class that had nothing to do with leap years, we could change the name of our module to `Leap` and mix it into a _class_ called `Year`.

If we want to make the three leap year rule methods private, we now have another choice of how to do it. We can use `module_function`.  `module_function` will make these methods available to the module, but when they get mixed into the class, they will be private. Module functions allow you to be selective with what can be called by the module itself, while still defining methods that can be mixed into other modules and classes.

```ruby
module Leap
  def leap?(year)
    mundane_leap?(year) || exceptional_century?(year)
  end

  def mundane_leap?(year)
    year % 4 == 0 && !century?(year)
  end

  def century?(year)
    year % 100 == 0
  end

  def exceptional_century?(year)
    year % 400 == 0
  end

  module_function :mundane_leap?, :century?, :exceptional_century?
end

class Year; extend Leap; end
```

Now, if we run the tests, they will all still pass, with the exception of the one we wrote that tries to call `exceptional_century?`

## Third Approach: Instance Methods

I want to stay with the idea that we might want a `Year` class that can do other things unrelated to determining whether we are talking about leap year or not. Really, there are a bunch of different years, but they all have certain things in common (which era they fall in, whether they are leap, etc.). In my mind, this would be a good use case for instances instead of class level methods.

What it look like if we brought the responsibility for knowing whether a year is leap back into a `Year` class, but passing that behavior onto _instances_ of year, instead of putting it on the eigenclass?

It's a little weird to be able to do something like `Year.new(2015).year`, so I'll name the state we're going to store `reckoning` instead.

```ruby
class Year
  def self.leap?(year)
    self.new(year).leap?
  end

  attr_reader :reckoning

  def initialize(reckoning)
    @reckoning = reckoning
  end

  def leap?
    mundane_leap? || exceptional_century?
  end

  private

  def mundane_leap?
    reckoning % 4 == 0 && !century?
  end

  def century?
    reckoning % 100 == 0
  end

  def exceptional_century?
    reckoning % 400 == 0
  end
end
```

The instance-based approach has certain advantages. We don't have to use anything exotic like `private_class_method`, `class << self`, or `module_function`, to make our methods private. This might make things easier to understand for future maintainers of our code.

We also gained access to the `reckoning`, that could be used to calculate other properites in the future. For example, if we wanted to write `to_julian` or `to_hebrew` methods, we'd already have the number we'd need to use to calculate that available to us.

Finally, we can now use a `protected` section as well. Methods in this section will only be visible to other instances of `Year`. We might use it to do something like this:

```ruby
class Year
  ...

  protected

  def ==(other)
    self.recoking == other.reckoning
  end

  ...
end
```

## Conclusion

We've looked at several different ways to write a program that can tell us whether a given year is a leap year. Each has its advantages and disadvantages.

We started with a class method that was a single-liner. There's a lot of value in the compactness of ` year % 4 == 0 && !(year % 100 == 0) || year % 400 == 0`. But it's also fairly hard to understand. If we decide to split some of that logic into separate methods, now we're faced with the problem of how to keep these new methods out of the interface of `Year`.

We used an eigenclass-based approach, simply hiding the implementation methods with `private_class_method`. We solved it using a module, treating the `leap?` method both as a mix-in and as a `module_function`. Finally, we pushed the functionality down to the instance level, which could help us in dealing with multiple `Year` objects down the road.

Which of these approaches is best?

It depends on lots of things: how you feel about privacy, how you expect the program to change in the future, how comfortable you are with scope and the more arcane methods in Ruby.  At the very least, it's nice to know what's available to you when thinking about what methods you want to expose, and what level of scope you should use it on.  I hope this post will factor into your thoughts when making these kind of decisions in the future.

P.S. What do you think? Did this make sense? Should I have used Structs or Lambdas? Throw us a comment below!

