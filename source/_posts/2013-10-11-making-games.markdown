---
layout: post
title: "Making Games"
date: 2013-10-11 09:03
---

My brain is completely saturated with code. For five weeks, I've lived, eaten, and breathed Ruby about 12 hours per day. I texted my brother (who's studying real estate) a few nights ago to tell him about it.

Me: "I'm up to my eyeballs in code."
Him: "I'm up to my eyeballs in water rights through the doctrine of prior appropriation and fee simple defeasible."

When pounding away at the same subject day in and day out, there comes a time when my brain starts to shut down, and I start to scream "NO MORE!!!" inside. I was just getting to that point as the week started, so I was relieved when Jeff announced that this would be a "Focus Week". Focus Week is a time when those who are behind can catch up, those who are progressing can solidify what they've learned, and those who are excelling can challenge themselves by exploring new things. It's also a time when everyone can take a breath; the pressure is off.

## Focus Week and Ruby Processing

Being in the "excelling" group at the current moment, I was given a simple assignment: choose a partner and build Tic Tac Toe using [Ruby Processing](https://github.com/jashkenas/ruby-processing) (RP). We'd gotten a brief introduction to RP last Friday as a fun activity, via the Jumpstart Lab [Process Artist Tutorial](http://tutorials.jumpstartlab.com/projects/process_artist.html). RP is a shim between Ruby and the [Java Processing](http://processing.org/), a language used to create graphics simulations. I dug a little deeper into RP over the weekend, using it to build a simple Mandelbrot fractal and attempting to animate it. Here's a still what I built by translating a Java [tutorial](http://processing.org/examples/mandelbrot.html) into Ruby:

{% img http://s9.postimg.org/8fghj6iin/Screen_Shot_2013_10_11_at_9_14_33_AM.png 'Mandelbrot Fractal' %}



## Tic Tac Toe

On Monday morning, I paired up with Rolen Le and we set out to build Tic Tac Toe, using a [Model View Controller](http://en.wikipedia.org/wiki/Model-view-controller) (MVC) framework. It was slow going at first. We were having a lot of trouble getting any file that contained "require 'ruby-processing'" to be able to talk to any other files via require. This meant that splitting our app into separate Model, View, and Controller files was not working. We compromised by put all the classes into a single file. But there was another problem:

We couldn't run tests.

So we didn't. It would have been difficult to run tests on the views of the app anyway, but once we threw up our hands and decided we couldn't test anything, we started down a very *dark* path. We wrote the whole app using no tests. Well, actually, we play tested. But it was terrible. Time after time, we would think that everything was working, then discover there was a weird bug. One of the funniest was: you could click on a square after it was taken, and it would change players even though no move was made. Thus, you could fill the whole board with just x's or o's. We eventually fixed all of the bugs. At least, I think we did. Without tests we'll never know.

At this point, the obvious thing to do would have been to write tests. But now that we'd whet our whistles on building games, we opted to split up and write some other games. Rolen headed off to build Chess, and I got started on Mancala.

## Mancala

Thankfully, Nathaniel Watts and Brian Winterling turned us on to the fact that you can use JRuby instead of Ruby 2.0 to get RP working a little better. Once I knew this, I was able to build Mancala with seperate files, and tests! The process was ***so*** much cleaner and easier with TDD. I built Mancala over three days, using an MVC framework. I also built extra classes for the pits and stores, and for the rules. My implementation uses the [Kalah Rules](http://boardgames.about.com/cs/mancala/ht/play_mancala.htm), but I wanted to make it so that you could write another rule set and use the same board to play with it. I ended up a small file that just runs RP, a model, two views, the ruleset (which is big controller), and a small controller to links them all together.

I was pretty happy with the result. I haven't refactored the code at all, so it's pretty messy. Next week, I'll probably try to clean it up. I'd also like to build custom graphics in Photoshop, like Nathaniel and Brian did for *their* [tic-tac-toe app](https://github.com/thewatts/tic-tac-toe).

My biggest takeaway from this game building adventure: TDD is *way* better than naked coding. There was so much messy, wasteful code in the first versions of Tic Tac Toe and Mancala. I was much happier with my results once JRuby enabled me to start using require again.

## On Groupings

Having been a teacher in the past, I'm interested in the teaching approaches Jumpstart Lab uses in gSchool. One thing that I saw work really well in my education career was grouping students together in a meaningful way. This week, we went from our usual whole class group to ability level groups. Although I like having the opportunity to talk and collaborate with other students at my level, I would not want to be ability grouped throughout the entire 24 weeks of gSchool. I enjoy having the chance to work with people of other skill levels, because it puts me into other roles: mentor, student, or just comrade on the path to being a Ruby ninja.

I experienced all kinds of grouping this week: the full class (at the beginning of the week and for the retrospective at the end), the challenge ability group of about eight students, pairing with Rolen, and working individually on Mancala. I think that changing up the setting like this is good for me. Being an introvert, I feel more comfortable with the smaller groupings than the full class setting. On the other hand, I think I get different benefits from being in groups of each size, and I wouldn't give up the full class environment if given the option.

## Web Applications

Next week we'll start building web applications. I'm pretty excited about it. Focusing on pure Ruby these past few weeks has really boosted my confidence in using the language, but I'm also ready for a change of focus. Exploring the world of HTTP requests, SQL, and Sinatra will be a nice change of pace. Outside of getting our Tic Tac Toe games to talk to each other this week (through threading and polling a simple server), I've never really worked with HTTP requests. My SQL experience is limited, and I know nothing about Sinatra.

It's my understanding that we'll also be expected to understand HTML, CSS, and a little bit of JavaScript to integrate the front and back-end parts of our app. I'm feeling glad that I spent some time going through tutorials on Codecademy, Code School, and Treebook before gSchool started; I already have some grounding in these languages. Next week, I'll be forced to recall my knowledge of HTML, CSS, JS, and SQL, and put it into practice *with* my newly learned back-end skills. I'm looking forward to integrating so many different languages toward a single goal (which is also the goal of gSchool): building web applications.

Our first web project will be using Sinatra to build [IdeaBox](http://tutorials.jumpstartlab.com/projects/idea_box.html), a simple system for recording ideas.

## Wrapping Up

I was grateful to have some time to reset my brain this week. It was starting to feel saturated. Playing with games and graphics has helped me feel fresh again. I plan to spend the upcoming weekend with the computer shut, reading a little of [HTML and CSS](http://www.amazon.com/gp/product/1118008189/ref=oh_details_o00_s00_i00?ie=UTF8&psc=1) by Jon Duckett, and coming back on Monday ready to dive headlong into building the web.

## Games

Here are the links to the games I built this week. You'll need JRuby with the ruby-processing gem installed to run them.

- [Tic Tac Toe](https://github.com/fluxusfrequency/tic-tac-toe.git)

- [Mancala](https://github.com/fluxusfrequency/mancala)
