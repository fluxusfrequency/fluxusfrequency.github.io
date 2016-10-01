---
title: 'Bring Back Your App: How To Prioritize Your Best Features'
date: 2015-10-21 02:52:44 Z
categories:
- source
layout: post
comments: true
---

This post originally appeared on [The Quick Left Blog](https://quickleft.com/blog/bring-back-app-prioritize-best-features/)

<img src="https://quickleft.com/wp-content/uploads/priritize-tasks-300x202.gif" alt="how to prioritize your best features"/>

## Introduction

The tech world moves fast. It's not uncommon for a startup to scale from two people in a coworking space to a team of thirty or more in a matter of months. Just as often, the team shrinks again due to sudden market changes or errant developers moving on to the next big thing.

When you're getting ready to rebuild your business, it can be hard to know where to begin. There are so many things to do, from marketing, to onboarding a new team, to deciding what to build next.

In [part one of this series](http://quickleft.com/blog/ramping-up-developers-on-code), we talked about some of things you can do on the technical side to get your dev team up and running with a minimum of fuss.

In this, the second and final part of this series, we'll focus on how to prioritize your best features. We'll look at how to decide what you should get rid of and what you should pull out into its own codebase. Then we'll tackle the question: what to build next?

## Finding Your Strategic Direction

Assuming your development team is all ready to go, you might start to ask yourself: "what do we build next?"

But before you assume that you're ready to start prototyping and shipping new features, you should take a hard look at what's already there. Then, ask yourself if you shouldn't do a little housekeeping first.

### Take An Inventory

It's a good idea to take an inventory of the workflows and features that are already in your application. Make a list of ways that users interact with the system. You could even consider [writing user stories](https://blog.engineyard.com/2015/happy-sad-evil-weird-feature-planning) for the functionality that's already there.

### Consider Removing Features

Here's a fun exercise. Once you've created a list of your use cases, go through and assign a value to each based on how important you think it is to your users. Then check out your analytics tool and compare your assumptions to the _actual_ engagement of each feature.

When you've completed this process, you can begin to reprioritize. Are there secondary features that aren't seeing much traffic? Maybe they were failed attempts to grow your user base that you never got around to pulling out. Maybe there are features that used to be popular, but a competing technology rendered them obsolete. Can you identify any parts of the app that require a lot of upkeep, but aren't really serving your users? Here's the tough question: what can you remove?

Think about [Basecamp](https://basecamp.com/). They used to be [37Signals](http://37signals.com/), but in February 2014, they announced that they were dropping support for all of their applications aside from Basecamp. They whittled it down to what they knew was successful so that they could focus all of their resources on it.

You can do the same thing on a smaller scale. Focus on the part(s) of your application that made the business a success in the first place. Do your marketing and landing pages drive users to interact with that feature? Is it easy to get to? Is it easy to _use_? Are there other features or workflows that are getting in the way of users finding their way there?

If you can identify some features that you can lose, you can focus your resources on the things that bring users to your site. You free up energy to refine them and make sure they're solid and bug-proof. It also gives you room to experiment with _new_ secondary features without overwhelming your users. I know one company with a well-established product that follows this rule: "you can't add a feature unless you remove one."

[Don't build yet](https://gettingreal.37signals.com/ch05_Start_With_No.php), even if you have customers clamoring for it. Remove the cruft first. Think of it as refactoring at an application level.

### Consider Splitting Out Services

In [part one of this series](http://quickleft.com/blog/ramping-up-developers-on-code), we talked a bit about refactoring your application code. If you do a little bit of refactoring, and find yourself getting into this whole code extraction thing, maybe you want to go whole hog. Talk with your engineers about whether it makes sense to extract an entire part of your app into a separate service. Some good candidates include: [exposing a REST API](http://fluxusfrequency.github.io/blog/2015/03/01/serving-custom-json-from-your-rails-api-with-activemodel-serializers/), [creating a gem](https://blog.engineyard.com/2014/wrapping-your-api-in-a-ruby-gem) to talk to your API, [storing data that's shared with other applications](https://quickleft.com/blog/how-to-create-and-expire-list-items-in-redis/), or integrating with an Elastic Search service.

In some cases, starting a whole new code base can take a lot of time, and might not be worth it. On the other hand, if it's hard to make changes in the code you've already got, sometimes extracting a service can actually speed things up. With a smaller surface area, and code written by people who _currently_ work for your company, it's easier for folks to grok what's going on. This leads to faster development.

If you do go down the road of extracting services, your team should take the time to define clear boundaries between the old app and the new one, and describe how they will interact in detail. For example, if you're pulling out an API, you might want to come up with a sample JSON response and the HTTP Status Codes that will be used _before_ you build it. Defining the limitations and structure beforehand can keep a project like this from dragging on endlessly.

### Use MVP To Figure Out Where To Go Next

A lot of people don't realize it, but a Minimum Viable Product (MVP) isn't just the first version of the thing you're building. It's actually a process. A way of testing out the market to find out whether things you _think_ that people will like are _actually_ things that people will like.

If you haven't tried driving your business this way, you should! Instead of just guessing what you should build next, or building whatever customers ask for, you can actually find out using research and numbers. Customer requests are great, but spiking on new features and getting information from analytics and customer feedback can really help you hit a home run.

Here's basic gist of MVP. You start with an idea. It might be an app. It might be a feature. Either way, you build _just enough_ to find out whether it makes sense to invest further in your idea. How do you find out? Measure user response with clicks and sign-ups that actually go nowhere, and use analytics, questionnaires, interviews, and emails. When you've got data, analyze it. Do they love it? Keep going! Was it a partial success? Ask yourself: what can I keep, what can I lose? Otherwise, pivot or abandon your idea. Build something else. Measure its success. Analyze your data. Repeat.

For a more specific walk-through of using MVP to drive your business direction, check out my [Actually MVP post](https://blog.engineyard.com/2015/actually-mvp).

Assuming you're getting ready to scale up your app, you probably already have some ideas about what you want to build. Stop for a second, and consider adding a couple of months of MVP-driven development to your roadmap instead.

## Wrapping Up

In this post, we looked at ways to find your strategic direction when beginning work on a product that's been on ice for a while. We talked about how to decide what to remove, what to extract, and what to build next. Hopefully, the process I've outlined here has got you thinking about how to prioritize your best features. Now, it's up to you to build _the next big thing_. Best of luck!

