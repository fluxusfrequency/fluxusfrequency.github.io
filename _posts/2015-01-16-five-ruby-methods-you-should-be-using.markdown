---
title: Five Ruby Methods You Should Be Using
date: 2015-01-16 13:55:21 Z
categories:
- source
layout: post
comments: true
---

This post was featured as one of Ruby Weekly's [Best Links of 2015](http://rubyweekly.com/issues/279).
Before that, it was the top featured article in [Ruby Weekly #229](http://rubyweekly.com/issues/229).
It was also the top featured article in issue #15.1 of the [Pointer.io](http://www.pointer.io/) newsletter.

It was originally published on [Engine Yard](https://blog.engineyard.com/2015/five-ruby-methods-you-should-be-using) and also appeared on the [Quick Left Blog](https://quickleft.com/blog/five-ruby-methods-you-should-be-using/).

## Introduction

There's something magical about the way that Ruby just flows from your fingertips. why once [said](http://mislav.uniqpath.com/poignant-guide/book/chapter-2.html) "Ruby will teach you to express your ideas through a computer." Maybe that's why Ruby has become such a [popular choice](http://blog.codeeval.com/codeevalblog/2014) for modern web development.

Just as in English, there are lots of ways to say the same thing in Ruby. I spend a lot of time reading and nitpicking people's code on [Exercism](http://exercism.io), and I often see exercises solved in a way that could be greatly simplified if the author had only known about a certain Ruby method.

Here's a look at some lesser-used methods solve a specific problem very well.

## Object#tap

Did you ever find yourself calling a method on some object, and the return value not being what you wanted it to? You were hoping to get back the object, but instead you got back some other value. Maybe you wanted to add an arbitrary value to a set of parameters stored in a hash. You update it with `Hash.[]`, but you get back `'bar` instead of the params hash, so you have to return it explicitly.

```ruby
def update_params(params)
  params[:foo] = 'bar'
  params
end
```

The `params` line at the end of that method seems extraneous.

We can clean it up with [Object#tap](http://www.ruby-doc.org/core-2.1.5/Object.html#method-i-tap).

It's easy to use. Just call it on the object, then pass `tap` a block with the code that you wanted to run. The object will be yielded to the block, then be returned. Here's how we could use it to improve `update_params`:

```ruby
def update_params(params)
  params.tap {|p| p[:foo] = 'bar' }
end
```

There are dozens of great places to use `Object#tap`. Just keep your eyes open for methods called on an object that don't return the object, when you wish that they would.

## Array#bsearch

I don't know about you, but I do a lot of looking through arrays for data. Ruby enumerables make it easy to find what I need: `select`, `reject`, and `find` are valuable tools that I use daily. But when the dataset is big, I start to worry about the length of time it will take to go through all of those records.

If you're using `ActiveRecord` and dealing with a `SQL` database, there's a lot of magic that happens behind the scenes to make sure that your searches are conducted with the least algorithmic complexity. But sometimes you have to pull all of the data out of the database before you can work with it. For example, if the records are encrypted in the database, you can't query them very well with `SQL`.

At times like these, I think hard about how to sift through the data with an algorithm that has the least complex *Big O* classification that I can. If you don't know about Big O notation, check out Justin Abrahms's [Big-O Notation Explained By A Self-Taught Programmer](https://justin.abrah.ms/computer-science/big-o-notation-explained.html) or the [Big-O Complexity Cheat Sheet](http://bigocheatsheet.com/).

The basic gist is that algorithms can take more or less time, depending on their complexity, which is ranked in this order: O(1), O(log n), O(n), O(n log(n)), O(n^2), O(2^n), O(n!). So we prefer searches to be in one of the classifications at the beginning of this list.

When it comes to searching through arrays in Ruby, the first method that comes to mind is `Enumerable#find`, also known as `detect`. However, this method will search through the entire list until the match is found. While that's great if the record is at the beginning, it's a problem if the record is at the end of a really long list. It takes O(n) complexity to run a `find` search.

There is a faster way. Using [Array#bsearch](http://www.ruby-doc.org/core-2.1.5/Array.html#method-i-bsearch), you can find a match with only O(log n) complexity. To find out more about how a Binary Search works, check out my post [Building A Binary Search](http://fluxusfrequency.github.io/blog/2014/01/31/building-a-binary-search/).

Here's a look at the difference in search times between the two approaches when searching a range of 50,000,000 numbers:

```ruby
require 'benchmark'

data = (0..50_000_000)

Benchmark.bm do |x|
  x.report(:find) { data.find {|number| number > 40_000_000 } }
  x.report(:bsearch) { data.bsearch {|number| number > 40_000_000 } }
end

         user       system     total       real
find     3.020000   0.010000   3.030000   (3.028417)
bsearch  0.000000   0.000000   0.000000   (0.000006)

```

As you can see, `bsearch` is much faster. However, there is a pretty big catch involved with using `bsearch`: the array must be sorted. This somewhat limits its usefulness, but it's still worth keeping in mind for occasions where it might come in handy, such finding a record in by a `created_at` timestamp that has already been loaded from the database.

## Enumerable#flat_map

When dealing with relational data, sometimes we need to collect a bunch of unrelated attributes and return them in an array that is not nested. Let's imagine you had a blog application, and you wanted to find the authors of comments left on posts written in the last month by a given set of users.

You might do something like this:

```ruby
module CommentFinder
  def self.find_for_users(user_ids)
    users = User.where(id: user_ids)
    user.posts.map do |post|
      post.comments.map |comment|
        comment.author.username
      end
    end
  end
end
```

You would then end up with a result such as:

```ruby
[[['Ben', 'Sam', 'David'], ['Keith']], [[], [nil]], [['Chris'], []]]
```

But you just wanted the authors! I guess we can call `flatten`.

```ruby
module CommentFinder
  def self.find_for_users(user_ids)
    users = User.where(id: user_ids)
    user.posts.map { |post|
      post.comments.map { |comment|
        comment.author.username
      }.flatten
    }.flatten
  end
end
```

Another option would have been to use `flat_map`.

This just does the flattening as you go:

```ruby
module CommentFinder
  def self.find_for_users(user_ids)
    users = User.where(id: user_ids)
    user.posts.flat_map { |post|
      post.comments.flat_map { |comment|
        comment.author.username
      }
    }
  end
end
```

It's not too much different, but better than having to call `flatten` a bunch of times.

## Array.new with a Block

One time, when I was in bootcamp, our teacher Jeff Casimir (founder of [Turing School](http://turing.io/)) asked us to build the game Battleship in an hour. It was a great exercise in object-oriented programming. We needed `Rule`s, `Player`s, `Game`s, and `Board`s.

Creating a represention of a `Board` is a fun exercise. After several iterations, I found the easiest way to set up an 8x8 grid was to do this:

```ruby
class Board
  def board
    @board ||= Array.new(8) { Array.new(8) { '0' } }
  end
end
```

What's going on here? When you call [Array.new](http://www.ruby-doc.org/core-2.1.5/Array.html#method-c-new) with an argument, it creates an array of that length:

```ruby
Array.new(8)
#=> [nil, nil, nil, nil, nil, nil, nil, nil]
```

When you pass it a block, it populates each of its members with the result of evaluating that block:

```ruby
Array.new(8) { 'O' }
#=> ['O', 'O', 'O', 'O', 'O', 'O', 'O', 'O']
```

So if you pass an array with eight elements a block that produces an array with eight elements that are all `'O'`, you end up with an 8x8 array populated with `'O'` strings.

Using the `Array#new` with a block pattern, you can create all kinds of bizarre arrays with default data and any amount of nesting.

## <=>

The [spaceship](http://www.ruby-doc.org/core-2.1.5/Object.html#method-i-3C-3D-3E), or sort, operator is one of my favorite Ruby constructs. It appears in most of the built-in Ruby classes, and is useful when working with enumerables.

To illustrate how it works, let's look at how it behaves for `Fixnums`.  If you call `5<=>5`, it returns `0`. If you call `4<=>5`, it returns `-1`. If you call `5<=>4`, it returns `1`. Basically, if the two numbers are the same, it returns `0`, otherwise it returns `-1` for least to greatest sorting, and `1` for reverse sorting.

You can use the spaceship in your own classes by including the `comparable` module and redefining `<=>` with logic branching to make it return `-1`, `0`, and `1` for the cases you want.

Why would you ever want to do that?

Here's a cool use of it I came across on Exercism one day. There's an exercise called Clock, where you have to adjust the hours an minutes on a clock using custom `+` and `-` methods. It gets complicated when you try to add more than 60 minutes, because that will make your minute value invalid. So you have to adjust by incrementing another hour and subtracting 60 from the minutes.

One user, `dalexj`, had a brilliant way to solve this, using the spaceship operator:

```ruby
  def fix_minutes
    until (0...60).member? minutes
      @hours -= 60 <=> minutes
      @minutes += 60 * (60 <=> minutes)
    end
    @hours %= 24
    self
  end
```

It works like this: until the minutes passed in are between 0 and 60, he subtracts either `1` or `-1` from the hours, depending on whether the minute amount is greater than 60. He then adjusts the minutes, adding either `-60` or `60` depending on the sort order.

The spaceship is great for defining custom sort orders for your objects, and can also come in handy for arithmetic operations if you remember that it returns one of three `Fixnum` values.

## Wrapping Up

Getting better at writing code is a process of learning. Since Ruby is a language, a lot of the time I spend trying to improve is spent of reading "literature" (i.e. code on Exercism andGitHub), and reading (what is essentially) the dictionary for my language: ([Rubydocs](http://ruby-doc.org/).

It's so much easier to write expressive code when you know more methods. I hope that this collection of curiosities helped expand your Ruby vocabulary.
