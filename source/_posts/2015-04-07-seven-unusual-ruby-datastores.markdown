---
layout: post
title: "Seven Unusual Ruby Datastores"
date: 2015-04-07 06:30:04 -0600
comments: true
categories:
---

This post appeared in [Ruby Weekly #240](http://rubyweekly.com/issues/240). It was also included in issue #27.1 of the [Pointer.io](http://www.pointer.io/) newsletter. It originally appeared on the [Engine Yard Blog](https://blog.engineyard.com/2015/seven-unusual-ruby-datastores)

## Introduction

Admit it: you like the unusual. We all do. Despite constant warnings against premature optimization, an emphasis on "readable code", and the old aphorism, "keep it simple, stupid", we just can't help ourselves. As programmers, we love exploring new things.

In that spirit, let's go on an adventure. In this post, we'll take a look at seven lesser-known ways to store data in the Ruby language.

## The Ones We Already Know

Before we get started, we'll set a baseline. What are the ways to store data in Ruby that we use every day? Well, these are the ones that come to mind for me: string, array, hash, CSV, JSON, and the filesystem.

We can skip all of these.

So what are some of the other ways to store data in Ruby? Let's find out.

## Struct

### What Is It?

A struct is a way of bundling together a group of variables under a single name. If you've done any C programming, you've probably come across structs before.

A struct is similar to a class. At its most basic, it's a group of bundled attributes with accessor methods. You can also define methods that instances of the struct will respond to.

In Ruby, structs inherit from Enumerable, so they come with all kinds of great behavior, like `to_a`, `each`, `map`, and member access with `[]`.

You can define a struct object by setting a constant equal to `Struct.new` and passing in some default attribute names. From there, you can create any number of instances of the struct, passing in attribute values for that instance.

Let’s explore one:

```ruby
Cat = Struct.new(:name, :breed, :hair_length) do
  def meow
    "m-e-o-w-w"
  end
end

tabby = Cat.new("Tabitha", "Russian Blue", "short")

tabby.name
=> "Tabitha"
tabby.meow
=> "m-e-o-w-w"
tabby[0]
=> "Tabitha"
tabby.each do |attribute|
  puts attribute
end
"Tabitha"
"Russian Blue"
"short"
=> #<struct Cat name="Tabitha", breed="Russian Blue", hair_length="short">
```

### When Would You Use It?

If you want to quickly define a class that has easily accessible attributes and little other behavior, structs are a great choice. Since they also respond to enumerable methods, they are great for use as stubs in tests.

If you want to stub a class and send it a message in a test, but you don't want to use [a double](https://relishapp.com/rspec/rspec-mocks/v/3-2/docs/verifying-doubles/using-an-instance-double), you can fake it with a struct in a single line of code.

Look how simple that is:

```ruby
fake_stripe_charge = Struct.new(:create)
```

Next we'll take a look at `Struct`'s close cousin, `OpenStruct`.

## OpenStruct

### What Is It?

An OpenStruct is somewhat like a hash. It's a data structure that you can use to store and access key-value pairs. In fact, it really _is_ a hash. Under the hood, each OpenStruct uses a hash for data storage. It also defines getters and setters automatically using `method_missing` and `define_method`.

There are three main differences between a struct and an open struct.

The first is that when you initialize a struct, you get back a class that inherits from `Struct`, which you must further instantiate, whereas calling `new` on an OpenStruct gives you back an `OpenStruct ` object.

Secondly, OpenStructs don't allow you to define behaviors by passing a block to the initializer as we did with the struct above.

Finally, OpenStructs must be passed an argument that responds to `each_pair` (such as a hash), whereas stucts expect a list of strings or symbols (to define their attribute names).

In the end, an OpenStruct is a much simpler than a struct.

OpenStruct lives in the Ruby Standard Library, so to use it in your code, you'll have to `require 'ostruct'`.

Let’s explore one:

```ruby
luke = OpenStruct.new({
  home: "Tatooine",
  side: :light,
  weapon: :light_saber
})

luke
=> #<OpenStruct home="Tatooine", side=:light, weapon=:light_saber>
luke.home
=> "Tatooine"
luke.side = :dark
=> :dark
luke
=> #<OpenStruct home="Tatooine", side=:dark, weapon=:light_saber>
```

### When Would You Use It?

As with Structs, I like to use OpenStructs as test stubs. Unfortunately, the metaprogramming used behind the scenes makes OpenStructs [much slower than hashes](http://ruby-doc.org/stdlib-1.9.3/libdoc/ostruct/rdoc/OpenStruct.html#class-OpenStruct-label-Implementation-3A), and they also respond to far fewer methods, so they aren't as flexible for everyday use. However, their built-in getters make them really useful anywhere that you need to inject an object that responds to a certain method.

## Marshalling

### What Is It?

Marshaling is way to serialize Ruby objects into a binary format. It converts them into a bytestream that can be saved and reconstituted later.

You marshal objects by calling `Marshal.dump` and `Marshal.load`.

Here’s an example:

```ruby
SpaceCaptain = Struct.new(:name, :rank, :affiliation)
=> SpaceCaptain

picard = SpaceCaptain.new("Jean-Luc Picard", "Captain", "United Federation of Planets")
=> #<struct SpaceCaptain name="Jean-Luc Picard", rank="Captain", affiliation="United Federation of Planets">

saved_picard = Marshal.dump(picard)
=> "\x04\bS:\x11SpaceCaptain\b:\tnameI\"\x14Jean-Luc Picard\x06:\x06ET:\trankI\"\fCaptain\x06;\aT:\x10affiliationI\"!United Federation of Planets\x06;\aT"
# Write to disk

loaded_picard = Marshal.load(saved_picard)
=> #<struct SpaceCaptain name="Jean-Luc Picard", rank="Captain", affiliation="United Federation of Planets">
```

### When Would You Use it?

There are plenty of use cases for serializing code running in memory and saving it for later reuse. For example, if you were writing a video game and you wanted to make it possible for a player to save their game for later, you could marshal the objects in memory (e.g. the player, her location in a map, and any enemies that are nearby) and persist them. You could then load them up again when the player is ready to continue.

Although there are other data serialization formats available, such as JSON, XML, and YAML (which we'll look at next), marshalling is [by far the fastest option](http://www.skorks.com/2010/04/serializing-and-deserializing-objects-with-ruby/) available in Ruby. That makes it particularly well-suited to situations where you dealing with large volumes of data or processing it at high speed.


## YAML

### What is It?

YAML, which stands for YAML Ain't Markup Language, is a widely-used format for serializing data in a human-readable format. It's available in many languages, of which Ruby is only one. The most widely-used Ruby YAML parser, [psych](https://github.com/tenderlove/psych), is a wrapper around `libyaml`, the C language parser.

YAML lives in the Ruby Standard Library, so to use it in your code, you'll have to `require 'yaml'`. You can use the [YAML::Store library](http://ruby-doc.org/stdlib-2.2.1/libdoc/yaml/rdoc/YAML/Store.html) to easily save data to disk.

Here’s an example of how to use that library:

```ruby
require 'yaml/store'

class Database
  DATABASE = YAML::Store.new('my_database')

  def self.save_person(user_data)
    DATABASE.transaction do
      DATABASE["people"] ||= []
      DATABASE["people"] << user_data
    end
  end
end

bilbo = {
  race: :hobbit,
  aliases: ["Bilba Labingi"],
  home: "The Shire",
  inventory: [:the_one_ring, :arkenstone]
}

Database.save_person(bilbo)
=> [{:race=>:hobbit, :aliases=>["Bilba Labingi"], :home=>"The Shire", :inventory=>[:the_one_ring, :arkenstone]}, {:race=>:hobbit, :aliases=>["Bilba Labingi"], :home=>"The Shire", :inventory=>[:the_one_ring, :arkenstone]}]
```

Here's what `my_database` would look like after running this code:

```yaml
---
people:
- :race: :hobbit
  :aliases:
  - Bilba Labingi
  :home: The Shire
  :inventory:
  - :the_one_ring
  - :arkenstone
```

### When Would You Use it?

YAML serves the same function as marshaling: it's a way to serialize Ruby objects for storage. It's quite a bit slower, but it's human-readable.

YAML is working behind the scenes when ActiveRecord is used to [serialize a record attribute](http://apidock.com/rails/ActiveRecord/Base/serialize/class) containing a hash or an array and save it to a text column in the database. When the attribute is retrieved, ActiveRecord deserializes it back from YAML into a Ruby object of its original data type.

## Set

### What Is It?

If you're familiar with mathematical [set theory](http://en.wikipedia.org/wiki/Set_theory), the `Set` class should be pretty intuitive. Sets respond to `intersection`, `difference`, `merge`, and many other Set operations.

It allows you to define a data structure that behaves like an unordered array that can only contain unique members. It exposes many of the same methods available when accessing arrays, but with a faster lookup. Like OpenStruct, Set uses hash under the hood.

Sets can be saved in [redis](http://redis.io/), which makes it possible to look them up very quickly.

Set lives in the Ruby Standard Library, so to use it in your code you'll have to `require 'set'`.

Here’s an example:

```ruby
require 'set'

basic_lands = Set.new
[:swamp, :island, :forest, :mountain, :plains].each do |land|
  basic_lands << land
end

basic_lands
=> #<Set: {:swamp, :island, :forest, :mountain, :plains}>

basic_lands << :swamp
# does nothing
=> #<Set: {:swamp, :island, :forest, :mountain, :plains}>

fires_lands = Set.new
[:forest, :mountain, :city_of_brass, :karplusan_forest, :rishadan_port].each do |land|
  fires_lands << land
end
fires_lands
=> #<Set: {:forest, :mountain, :city_of_brass, :karplusan_forest, :rishadan_port}>

basic_lands.intersection(fires_lands)
=> #<Set: {:forest, :mountain}>

basic_lands.difference(fires_lands)
=> #<Set: {:swamp, :island, :plains}>

basic_lands.subset?(fires_lands)
=> false

basic_lands.merge(fires_lands)
=> #<Set: {:swamp, :island, :forest, :mountain, :plains, :city_of_brass, :karplusan_forest, :rishadan_port}>
```

### When Would You Use it?

Sets are great for situations where you need to make sure that a given element isn't contained in a collection more than once. For example, if you were using tags in an application that was not backed by a database.

They're also great for comparing the equality of two lists without caring about their order (as an array would). You could use this feature to check whether the data stored in memory is in sync with another collection fetched from a remote server.


## Queue

### What Is It?

A [Queue](http://ruby-doc.org/core-2.2.1/Queue.html) is a place that can be used hold values that you want to share between threads. It's basically a stack that is visible to all of the concurrently running thread process in a given Ruby environment.

If you want to limit the amount of data that can be shared, you can use a [SizedQueue](http://ruby-doc.org/core-2.2.1/SizedQueue.html).

Here’s an example:

```ruby
require 'thread'
chess_moves = Queue.new

player_moves = Thread.new do
  chess_moves << "e4"
  sleep(1)
  chess_moves << "e5"
  sleep(1)
  chess_moves << "f4"
end

game_board = Thread.new do
  while chess_moves.length > 0
    move = chess_moves.pop
    # update the ui with the move
  end
end
```

### When Would You Use it?

Queues are extremely helpful in any application that runs code concurrently. For example, background processing libraries like [Redis](https://github.com/resque/resque) [make use of a queue](https://github.com/resque/resque/blob/master/lib/resque/queue.rb) to check for the latest jobs and instruct workers to run them.

## ObjectSpace

### What Is It?

The ObjectSpace module is a collection of methods that can be used to interact with all of the living objects in the current Ruby environment, as well as the garbage collector.

You can use it to check out all of the objects currently living in memory, look up objects by the object ID reference, and trigger garbage collector runs. You can also define a hook to be triggered when any object of a given class is removed from the ObjectSpace using `ObjectSpace#define_finalizer`.

How is ObjectSpace a data store? Well, it's the highest-level data store (that hasn't been interpreted or compiled yet) in any place that you can run Ruby code. Any time you define or remove an object from memory, you are changing what is visible in the ObjectSpace.

Let's take a look at everything that's available in an IRB session.

```ruby
object_counts = Hash.new(0)
ObjectSpace.each_object do |o|
  object_counts[o.class] += 1
end

require "pp"
pp object_counts

{
  String=>67073,
  Array=>14474,
  Regexp=>164,
  Gem::Specification=>299,
  Hash=>1023,
  # and many more...
}
```

If we create a new object, it ends up in the ObjectSpace.

```ruby
require "ostruct"

ObjectSpace.each_object(OpenStruct).count
=> 0
spidey = OpenStruct.new({ name: "Peter Parker", species: "Human Mutate" })

ObjectSpace.each_object(OpenStruct).count
=> 1
```

### When Would You Use it?

Although you already use the ObjectSpace all the time, whether you realize it or not, knowing about the methods it exposes opens up a lot of possibilities for investigating and improving the performance of your code.

The best use I've seen for ObjectSpace so far is using it to detect memory leaks. [This article](https://cirw.in/blog/find-references) shows an interesting way to map the objects in your object space to create a graph that is useful in tracking down and fixing memory leaks.



## Conclusion

Ruby is such a fun language to write because there are so many ways to say the same thing. It doesn't stop at writing statements and expressions, though. You can also store data in a huge number of ways.

In this post, we looked at seven fairly unusual ways to handle data in Ruby. Hopefully, reading through them has given you some ideas for how to handle persistence or in-memory storage in your own applications.

Until next time, happy coding!

P. S. We know that there are other unusual datastores out there. What are some of your favorites and how do you use them? Leave us a comment!

