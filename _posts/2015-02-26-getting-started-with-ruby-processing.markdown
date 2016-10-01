---
title: Getting Started With Ruby Processing
date: 2015-02-26 13:20:55 Z
categories:
- source
layout: post
comments: true
---

This post was a featured article in [Ruby Weekly #234](http://rubyweekly.com/issues/234).

It was originally published on [Engine Yard](https://blog.engineyard.com/2015/getting-started-with-ruby-processing).

# Introduction

If you're like me, you love to code because it is a creative process. In another life, I am a musician.

I've always loved music because it represents a synthesis of the measurable concreteness of math and the ambiguity of language. Programming is the same way.

But desipite the creative potential of programming, I often myself spending my days working out the kinks of HTTP requests or dealing with SSL certificates. Some part of my yearns for a purely [Apollonion](http://en.wikipedia.org/wiki/Apollonian_and_Dionysian) environment in which to use code to make something new and unseen.

When I feel a void for purely creative coding, I turn to the [Processing language](https://processing.org/). Processing is a simple language, based on Java, that you can use to create digital graphics. It's easy to learn, fun to use, and has an amazing online community comprised of programmers, visual artists, musicians, and interdiscplinary artists of all kinds.

In 2009, [Jeremy Ashkenas](https://twitter.com/jashkenas), creator of Backbone.JS, Underscore.JS, and Coffeescript), published the [ruby-processing gem](https://rubygems.org/gems/ruby-processing). It wraps Processing in a "thin little shim" that makes it even easier to get started as a Ruby developer. In this post, we'll take a look at how you can create your first interactive digital art project in just a few minutes.

## What Is Processing?

Processing is programming language and IDE built by [Casey Reas](http://reas.com/) and [Benjamin Fry](http://benfry.com/), two protegés of indisciplinary digital art guru [John Maeda](http://www.maedastudio.com/) at the MIT Media Lab.

Since the project began in 2001, it's been helping teach people to program in a visual art context using a simplified version of Java. It comes packaged as an [IDE](https://processing.org/download/) that can be downloaded and used to create and save sketches.

## Why Ruby Processing?

Since Processing already comes wrapped in an easy-to-use package, you may ask: "why should I bother with Ruby Processing?"

The answer: if you know how to write Ruby, you can use Processing as a visual interface to a much more complex program. Games, interactive art exhibits, innovative music projects, anything you can imagine; it's all at your fingertips.

Additionally, you don't have to declare types, `void`s, or understand the differences between `float`s and `int`s to get started.

Although there are some drawbacks to using Ruby Processing, most notably slower performance, having Ruby's API available to translate your ideas into sketches more than makes up for it.

## Setup

When getting started with Ruby Processing for the first time, it can be a little bit overwhelming to get all of the dependencies set up correctly. The gem relies on JRuby, Processing, and a handful of other things. Here's how to get them all installed and working.

I'll assume you already have the following installed: homebrew, wget, java, and a ruby manager such as rvm, rbenv or chruby.

### Processing

[Download Processing](https://www.processing.org/download/) from the official website and install it.

When you're done, make sure that the resulting app is located in your `/Applications` directory.

### JRuby

Although it's possible to run Ruby Processing on the [MRI](http://en.wikipedia.org/wiki/Ruby_MRI), I highly suggest using JRuby. It works much better, since Processing itself is built on Java.

Install the latest JRuby version (1.7.18 at the time of this writing). For example, if you're using `rbenv`, the command would be `rbenv install jruby-1.7.18`, followed by `rbenv global jruby-1.7.18` to set your current ruby to JRuby.

### Ruby Processing

Install the `ruby-processing` gem globally with `gem install ruby-processing`.
If you're using `rbenv`, don't forget to run `rbenv rehash`.

### JRuby Complete

You'll need the `jruby-complete` Java jar. Fortunately, there are a couple of built-in Ruby Processing commands that make it easy to install. `rp5` is the Ruby Processing command. It can be used to do many things, one of which is to install `jruby-complete` using `wget`. To do so, run:

```
rp5 setup install
```

Once it's complete, you can use `rp5 setup check` to make sure everything worked.

### Setup Processing Root

One final step. You'll need to set the root of your Processing app. This one-liner should take care of it for you:

`echo 'PROCESSING_ROOT: /Applications/Processing.app/Contents/Java' >> ~/.rp5rc`

### Ready To Go

Now that we have everything installed and ready to go, we can start creating our first piece of art!


## Making Your First Sketch

There are two basic parts to a Processing program: `setup` and `draw`.

The code in `setup` runs one time, to get everything ready to go.

The code in `draw` runs repeatedly in a loop. How fast is the loop? By default, it's 60 frames per second, although it can be limited by your machine's processing power. You can also manipulate it with the `frame_rate` method.

Here's an example sketch that sets the window size, background and stroke colors, and draws a circle with a square around it.

```ruby
def setup
  size 800, 600
  background 0
  stroke 255
  no_fill
  rect_mode CENTER
end

def draw
  ellipse width/2, height/2, 100, 100
  rect width/2, height/2, 200, 200
end
```

Here's a quick run-through of what each of these methods is doing:

-`size()`: Sets the window size. It takes two arguments: width and height (in pixels).
-`background()`: Sets the background color. It takes four arguments: R, G, B, and an alpha (opacity) value.
-`stroke()`: Sets the stroke color. Takes RGBA arguments, like `background()`.
-`no_fill()`: Tells Processing not to fill in shapes with the fill color. You can turn it back on with `fill()`, which takes RGBA values.
-`rect_mode`: Tells Processing to draw rectangles using the x and y coordinates as a center point, with the other two arguments specifying width and height. The other [available modes](https://processing.org/reference/rectMode_.html) are: CORNER, CORNERS, and RADIUS.
-`ellipse`: Draws an ellipse or circle. Takes four arguments: x-coordinate, y-coordinate, width, and height.
-`rect`: Draws a rectangle or square. Takes four arguments: x-coordinate, y-coordinate, width, and height.

Note that the coordinate system in Processing starts at the top-left corner, not in the middle as in the Cartesian Coordinate System.

## Running the Program

If you're following along at home, let's see what we've made! Save the code above into a file called `my_sketch.rb`.

There are two ways to run your program: you can either have it run once with `rp5 run my_sketch.rb`, or you can watch the filesystem for changes with `rp5 watch my_sketch.rb`. Let's just use the `run` version for now.

<img src="http://i.imgur.com/VhQo15W.png"/>

Pretty basic, but it's a good start! Using just the seven methods above, you can create all kinds of sketches.

## Other Commonly Used Methods

Here are a few other useful Processing methods to add to your toolbox:

-`line()`: Draws a line. Takes four argments: x1, y1, x2, y2. The line is drawn from the point at the x, y coordinates of the first two arguments to the point at the coordinates of the last two arguments.
-`stroke_weight()`: Sets the width of the stroke in pixels.
-`no_stroke()`: Tells Processing to draw shapes without outlines.
-`smooth()`: Tells Processing to draw shapws with anti-aliased edges. On by default, but can be disables with `noSmooth()`.
-`fill()`: Sets the fill color of shapes. Takes RGBA arguments.

For a list of all the methods available in vanilla Processing, check out [this list](https://www.processing.org/reference/). Note that the Java implementation of these methods is in `camelCase`, but in Ruby they are _probably_ in `snake_case`.

[Some methods have also been deprecated](https://github.com/jashkenas/ruby-processing/wiki/Deprecation-of-methods), usually because you can use Ruby to do the same thing more easily.

If you see anything in the Processing docs and can't get it to run in Ruby Processing, use `$app.find_method("foo")` to to search the method names available in Ruby Processing.

## Responding to Input

Now that we know how to make a basic sketch, let's build something that can respond to user input. This is where we leave static visual art behind, and start to make interactive digital art.

Although you can use all kinds of physical inputs to control Processing (e.g. Arduino, Kinect, LeapMotion), today we'll just use the mouse.

Processing exposes a number of variables exposing its state at runtime, such as `frame_count`, `width`, `height`. We can use the `mouse_x` and `mouse_y` coordinates to control aspects of our program.

Here's a sketch based on the `mouse_x` and `mouse_y` positions. It draws lines of random weight starting at the top of the screen at the mouse's x position (`mouse_x, 0`) to a y coordinate between 0 and 200 pixels to the right of the mouse's y position (`mouse_y + offset, height`).

```ruby
def setup
  size 800, 600
  background 0
  stroke 255, 60 # first argument is grayscale value, second is opacity
  frame_rate 8
end

def draw
  r = rand(20)
  stroke_weight r
  offset = r * 10
  line mouse_x, 0, mouse_y + offset, height
end
```


Load that up and check it out!

<img src="http://i.imgur.com/HkS0XiH.png" />

## Wrapping Your Sketch in a Class

One last not before we go: you can totally call other methods from within your `setup` and `draw` methods. In fact, you can even wrap everything in a class that inherits from `Processing::App`.

You can do everything you normally do in Ruby, so you can build a whole project, with logic branches and state, that controls the visual effect through these two methods.

Here's a snippet from a version of Tic Tac Toe I built with [Rolen Le](https://twitter.com/RolenTLe) during gSchool.

```ruby
require 'ruby-processing'

class TicTacToe < Processing::App
  attr_accessor :current_player

  def setup
    size 800, 800
    background(0, 0, 0)
    @current_player = 'x'
  end

  def draw
    create_lines
  end

  def create_lines
    stroke 256,256,256
    line 301, 133, 301, 666
    line 488, 133, 488, 666
    line 133, 301, 666, 301
    line 133, 488, 666, 488

    #borders
    line 133, 133, 666, 133
    line 666, 133, 666, 666
    line 133, 666, 666, 666
    line 133, 133, 133, 666
  end

  ...
end
```

To see the rest of the code, visit the [GitHub repo](https://github.com/fluxusfrequency/tic-tac-toe).

Another example of a game I built early on in my programming career can be found [here](https://github.com/fluxusfrequency/mancala). I later did a [series](http://fluxusfrequency.github.io/blog/2014/01/17/refactoring-2-replace-method-with-method-object/) of [refactorings](http://fluxusfrequency.github.io/blog/2014/01/18/refactoring-3-introduce-named-parameter-and-introduce-class-annotation/) of this [code](http://fluxusfrequency.github.io/blog/2014/01/27/refactoring-4-remove-middle-man/) on my [personal blog](http://fluxusfrequency.github.io/blog/archives/).

I'm still working on a pattern for game development with that I like. Keep an eye out for future posts about the best way to build a game with Ruby Processing.

## Learning More

There's so much more you can do in Processing than what we've covered here! Bézier curves, translations, rotations, images, fonts, audio, video, and 3D sketching are all available.

The best way to figure out how to do a lot of sketching. Just tinkering with the methods covered in this post would be enough to keep you busy creating new things for years.

If you've really caught the bug and want to go even _deeper_, check out some of these resources to learn more.

### Built-in Samples

If you run `rp5 setup unpack_samples`, you'll get a bunch of Processing sketch samples in a directory located at `~/rp_samples`. I encourage you to open them up and take a look. There's a lot you can glean by changing little bits of code in other projects.

### Online Examples From Books

Learning Processing is an excellent book by Daniel Shiffman. In addition to being a valuable resource for Processing users, it has a number of [examples available online](http://www.learningprocessing.com/examples/).

Daniel Shiffman also wrote a book called [The Nature of Code](http://natureofcode.com/). The examples from it have been [ported to Ruby](https://github.com/ruby-processing/The-Nature-of-Code-Examples-in-Ruby) and are another great resource for learning more.

### Process Artist

There's a great Jumpstart Lab tutorial called [Process Artist](http://tutorials.jumpstartlab.com/projects/process_artist.html), that walks you through building a drawing program à la MSPaint.


## Conclusion

Processing is an awesome multi-disciplinary tool. It sits at the intersection of coding, visual art, photography, sound art, and interactive digital experiences. With the availability of Ruby Processing, it's super easy to get started.

If you're a programmer looking for a way to express your creativity, you couldn't find a better way to do it than to try tinkering with Processing. I hope this post gets you off to a great start. Good luck and keep sketching!

