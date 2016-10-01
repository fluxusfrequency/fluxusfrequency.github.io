---
title: Web Applications
date: 2013-10-18 09:01:00 Z
categories:
- source
layout: post
---

This week we dug in to building a web application for the first time in gSchool. After six weeks of nothing but pure Ruby (well, almost pure - there was some Command Line, Git, and Javascript in there), it was a nice change of pace to see the language of the web again. I haven't used HTTP requests, HTML tags, or CSS selectors in months, so this week felt like a breath of fresh air.

We've begun learning to use Sinatra to build light weight Rack applications. Knowing that Sinatra was on the menu, I spent last weekend working through [Jump Start Sinatra](http://www.sitepoint.com/store/jump-start-sinatra/) by Darren Jones. The book walks you through building Songs By Sinatra, a light weight web app that catalogs the songs of Frank Sinatra. Along the way, it introduces the reader to Ruby Slim templating, flash messages, DataMapper, AJAX, a little bit of jQuery, and deploying to Heroku. I really enjoyed the project. It was small and quick to complete, but touched on many topics I've been wanting to explore.

Here is a link to my completed version of [Songs By Sinatra](http://songs-by-sinatra-flux.herokuapp.com/).

Jeff started the week with a morning lecture on the structure of the internet. He covered HTTP requests, IP addresses, DNS lookup, the DNS root name servers, and the structure of the DNS system. From there, he explained Distributed Denial of Service (DDoS) attacks. These are a big topic in computer security right now. The most common type of DDoS is a "hacking" attack in which someone controls a "botnet": hundreds or thousands of computers infected with backend malware. The botnet allows the attacker to use the zombie computers' processing power to carry out an attack on a certain target. At the designated time, the attacker directs all of the computers to repeatedly send requests to the target. The target is then drowned in requests, and becomes unreachable.

After covering DDoSs, we talked about many other aspects of the internet's dark underbelly, including [The Silk Road](http://en.wikipedia.org/wiki/Silk_Road_%28marketplace%29) and other hidden services accessible through the [Tor](https://www.torproject.org/) network, as well as [Bitcoins](http://vimeo.com/63502573). It was all fascinating stuff, all very cloak and dagger!

Once the week was underway, we dug into Sinatra with the Jumpstart Lab tutorial for [WebGuesser](http://tutorials.jumpstartlab.com/projects/web_guesser.html). It was a nice easy project that eased us in to using GET, PUT, POST, and DELETE requests via an app built in Sinatra.

{% img center http://s9.postimg.org/3mw3ucrrj/Screen_Shot_2013_10_18_at_9_21_56_PM.png 'My IdeaBox In Production' 'A website in the process of being built' %}

Soon after that we were introduced to our current project: [IdeaBox](http://tutorials.jumpstartlab.com/projects/idea_box.html). I've dug in hard this week. So far, I've completed the basic project as well as the tagging, statistics, and search extensions. I've been playing with some templating. After getting frustrated with ERB, I decided to try HAML for a while, but ended up going back to Slim, which I find much more intuitive. Having few design skills, I decided to use a CSS library for the front end. I'd already worked with Bootstrap in the past, so I decided to give [Zurb Foundation](http://foundation.zurb.com/) a go. So far, it's been easy to get it up and running, but it's sometimes hard to make the javascript elements display the way they look in the documentation. At any rate, I'm having a great time with the project.

Compared to building command line applications, I'm enjoying web apps more. The main reason is that it gives me a chance to switch gears. When I get tired of just dealing with Ruby code, I can go work on the front end. When that gets tiring, I head back to the model and do some more logic work.

## Work Habits

For the most part, I've been able to keep up my good work habits this week.

On the one hand, I've found it a lot more difficult to write effective tests in this environment. Whenever run my application test, it starts up the server. That can't be right, I have to CTL+C it every time. Also, I'm tempted not to write enough tests, especially route tests, when using Sinatra. But after last week's test-less mayhem with Tic Tac Toe, I'm fully convinced I need them. I have yet to try Capybara; perhaps that will help.

On the other hand, I've actually been a lot better about staying true to my Pomodore timer. Although my classmate Luke gave a Lightning Talk blasting the Pomodore method, I've found that I get really weird when I don't take regular breaks. With my old timer, [Menubar Countdown](http://www.makeuseof.com/tag/menubar-countdown-is-a-mac-timer-app-that-talks-to-you/), I would often ignore the end of my session and keep working. Now I'm using [Tadam](http://www.tadamapp.com), which puts a big black box on the screen and forces you to take a break. So far, the switch has helped keep me honest. I've also committed myself to taking walks outside. It really helps keep me from feeling like I'm in a cave/tunnel/timewarp all the time.

## Working Individually

For the IdeaBox project, we have small working groups. Although each person is completing the project individually, we have a cohort to turn to when we encounter problems or want to think through solutions. For me, this is a very comfortable apporoach. I tend to be a very independent, resourceful learner, but I'm not independent to the point of being a recluse. I've been enjoying helping team mates, and pairing with them a little bit. But most of all, I like talking through ideas and approaches to problems with them.

## Assessments Next Week

Next week, we'll be going through an assessment with one of the instructors. I'm not nervous about this, because there is nothing on the line. A failure on the assessment would mean little in terms of my coding eduction. On the contrary, I'm excited to have someone looking over my shoulder as I write. When I'm working on a project, I try to solve problems on my own before asking an instructor for help. Although this gives me confidence because I know I can work things out on my own, I also wonder if there are approaches and work flow improvements that I'm not seeing, because I'm the only one hearing my thought process with me. I hope that the assessment will be a chance for Jeff, Franklin or Katrina to watch how I solve problems, and make suggestions for efficiencies I may not be seeing as I go.

## Moving Forward

Last Wednesday, I realized that I was spending a lot of time building extra features at the expense of basic functionality, and going off on learning tangents that weren't helping me make any progress on the project. I decided to imagine where I would like the project to be on Thursday morning, when the project is due. I split up the remaining work to a daily schedule. I hope that by sticking to it I will have all of the features done on schedule, with some time for refactoring and bug fixes built in. I'm really excited to be building a web app on both the front and back end for the first time. Even though it is just a tutorial, I'm proud of what I'm building, and I can't wait to share it with the world!

