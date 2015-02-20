---
layout: post
title: "Building A Ruby List Comprehension"
date: 2014-11-03 06:40:09 -0600
comments: true
categories:
---

This post was a featured article in [Ruby Weekly](http://rubyweekly.com/issues/221).

It was originally published on [Engine Yard](https://blog.engineyard.com/2014/ruby-list-comprehension) and appeared on the [Quick Left Blog](https://quickleft.com/blog/building-a-ruby-list-comprehension/).

## Introduction

As developers, we're in the business of continually bettering ourselves. Part of that process is pushing ourselves to learn and use better code patterns, try new libraries, and pick up new languages. For me, the latest self-learning project has been picking up Python.

As I’ve worked with it, I’ve discovered the joy of [list comprehensions](https://docs.python.org/3.5/tutorial/datastructures.html#list-comprehensions), and I’ve been wondering what it would take to implement a similar syntax in Ruby. I decided to give it a try. This exercise yielded several insights into the inner workings of Ruby, which we'll explore in this post.

## Snake Handling

<img src="//quickleft-production.s3.amazonaws.com/uploads/asset/attachment/194/asset.gif">

I’m primarily a Rubyist. I’ve always enjoyed the natural way that Ruby flows off the fingers, and heard that Python was similar. It sounded easy enough, especially since [this article](http://www.stavros.io/tutorials/python/) promised I could learn Python in ten minutes.

It took a little while to get used to some of the differences in Python, like capitalizing booleans and passing `self` into all of my methods--but they were mostly superficial differences. As I solved more and more Python problems on [exercism.io](http://exercism.io/), the syntax began to feel natural.  I felt like everything I wrote in Python had an analog in Ruby. Except list comprehensions.

## You Cannot Grasp The True Form Of Lists

In Python, Coffeescript, and many functional languages, there is a really cool operation you can do called a _list comprehension_. It's something like a `map`, but it's written in mathematical [set builder notation](http://en.wikipedia.org/wiki/Set-builder_notation), which looks like this:

Instead of doing this:

```
(1..100).map { |i| 2 * i if i.even? }
```

You do this:

```
[2 * x for x in xrange(1, 100)]
```

You can even nest comprehensions:

```
return [[print(str(x) for x in y] for 2 * y in xrange(1, 100)]
```

These are so cool, in fact, that I decided to see if I could implement them in Ruby.

## How It Works

<img src="//quickleft-production.s3.amazonaws.com/uploads/asset/attachment/193/asset.gif">

My first thought in writing a list comprehension in Ruby was to make it look like Python. I wanted to be able to write square brackets with an expression inside of them. To make that happen, I would need to override the `main` `[]()` method. It turns out that that method is an alias for the `Array#[]` [singleton method](http://www.ruby-doc.org/core-2.1.3/Array.html#method-c-5B-5D).  It's not so easy to monkey patch, because you can't really call `super` on it.

I decided to abandon this approach and create a new method called `c()`, that I would put in a module and include in `main`.  Ideally, this method would take a single argument with this syntax: `x for x in my_array if x.odd?`. Since you can't really pass an expression like this into a method and expect Ruby to parse it properly, I opted to pass it in as a string and parse it. I was into this idea, but not really interested in rewriting the Ruby source code.

## Caught In A Loop

<img src="//quickleft-production.s3.amazonaws.com/uploads/asset/attachment/192/asset.gif">

My first goal was to get the basic `x for x in my_array` part working.

I wrote a test:

```
class ComprehensionTest < Minitest::Test
  def setup
    extend ListComprehension
  end

  def test_simple_array
    result = c('n for n in [1, 2, 3, 4, 5]')
    assert_equal [1, 2, 3, 4, 5], result
  end
end

```

This was fairly straightforward to get passing.

```
module ListComprehension
  def c(comprehension)
    parts = comprehension.split(' ')
    if parts[0] == parts[2]
      eval(comprehension.scan(/\[.*\]/).last)
    end
  end
end

```

I continued working along, refactoring to have the module instantiate a `Comprehension` class and evaluate parts of the string:

```
module ListComprehension
  def c(expression)
    Comprehension.new(expression).comprehend
  end
end

class Comprehension
  def initialize(expression)
    @expression = expression
  end

  def comprehend
    # Do some parsing and call `eval` on some stuff
  end
end

```

But pretty soon, I ran into a problem.

## No Scope

<img src="//quickleft-production.s3.amazonaws.com/uploads/asset/attachment/191/asset.gif">

When I defined a variable in the test scope, then tried to evaluate it in the `comprehension` argument string, my Comprehension couldn't access it.

For example, I wrote this test:

```
def test_array_variable
  example = [1, 2, 3, 4, 5]
  result = c ('x for x in example)
  assert_equal [1, 2, 3, 4, 5], result
end
```

When I tried to call `eval` on `example`, I got:

```
NameError: undefined local variable or method `example' for #<Comprehension:0x007f9cec989fc8>
```

So how could I acceess the scope where `example` was defined? I did some googling, and discovered that you can access the calling scope a lot more easily from block than a string that you pass into a method.

With that in mind, I changed the interface of `Comprehension` to take a block instead of a string. To call `c()`, you would now write `c{ 'x for x in example' }` instead of `c('x for x in example')`.

Inside of the `Comprehension` class, I did:

```
class Comprehension
  def initialize(&block)
    @comprehension = block.call
    @scope = block.send(:binding)
  end

  ...

end
```

Now I could call `eval` on the calling scope by doing:

```
def comprehend
  # some parsing
  collection = scope.send(:eval, parts.last)
  # carry out some actions on the collection
end
```

I had no idea you could access the calling scope like this in Ruby. It opened my eyes to the whole world of accessing callers and callees in a way that I normally don't think about in Ruby.

## You Obviously Worked Hard On These Plans, Kernel Klink

<img src="//quickleft-production.s3.amazonaws.com/uploads/asset/attachment/190/asset.gif">

I wasn't all that happy with having to define my `c()` method in a module and include it in the `main` scope. I really just wanted it to be available automagically if you required `comprehension.rb`.

After poking around a bit, I found an [article](http://hopsoft.github.io/blog/ruby-metaprogramming-idioms/) on metaprogramming that showed me how you can monkey patch `Kernel` itself.

After changing `module ListComprehension` to `module Kernel`, I was able to remove the setup method entirely from my test suite. I didn't realize it was this easy to get methods defined in the `main` scope. Even though it is probably very wrong in many situations, it's cool to get an understanding of how Ruby itself is put together.

## Lessons Learned

I set out to write a list comprehension in Ruby, and in a way, I failed.  I was hoping to be able to write an expression inside of square brackets and have Ruby parse it. I ended up settling for a string inside of a block instead.

What's more, my comprehension implementation is lacking several features. It doesn't support method calls in the conditional, so you can't write `c{'x for x in (0..10) if Array(x)'}`. You can't pass arguments to the conditional either, so you can't do `c{'x for x in (1..10) if x.respond_to?(:even?)'}`. You can't access an index while you're looping. And perhaps most disappointing of all, you can't nest comprehensions.

But despite these shortcomings, I felt like this exercise was a great success, because I learned three things:

1. When you call `[]` to create an array, you're really calling `Array#[]`, which delegates to `Array#new`.

2. You can access the calling scope of a block with `block.send(:binding)`.

3. You can monkey patch `module Kernel` to get methods available in `main`.

To the the knowledge gained from this adventure was totally worth it.  Although I did not create a library I would expect people to use, I learned a lot about how Ruby works, and had a great time solving the problem. To check out the full results, please [visit my GitHub profile](https://github.com/fluxusfrequency/ruby-comprehension.git).

P.S. How would you have solved it? Maybe you've written a natively parsed list comprehension yourself? If so, tweet at me @fluxusfrequency.

