---
title: Actually MVP
date: 2015-05-13 12:11:09 Z
categories:
- source
layout: post
comments: true
---

This post originally appeared on [Engine Yard](https://blog.engineyard.com/2015/actually-mvp).

## Introduction

In the startup world, there is a lot of talk about building _[Minimum Viable Products](http://en.wikipedia.org/wiki/Minimum_viable_product)_ (MVPs). At this point, the concept has become so well-accepted that it has almost become a kind of unquestioned dogma. Yet there is a lot of disagreement about what MVP is exactly, and how to carry it out. Many people in the software industry assume that they know what MVP means, and claim to be using the process, but their production workflow tells a different story.

When it comes to building software, it is often tempting to take an approach akin to building a skyscraper: write the blueprints, obtain the necessary prerequisites, then build it to spec. But software is a quickly shifting market. A businessperson may think she knows what the market wants, and plan and begin a project to meet that desire. But by the time the product is built, the needs of consumers have often morphed in a direction that she could never have foreseen.

In this post, we'll take explore some common misconceptions about MVP, some different ways to approach building one with software, and how to best use this tool if you're the CEO or CTO of a startup, a product manager for an established company, or a consultant.

## Why MVP?

We hear a lot of talk about MVP and its value, but as a businessperson, why should you care? The reason is simple: it prevents you from spending money building a product that nobody wants.

When you build your business around small, successive iterations, the time before you can reflect on lessons learned is as small as possible. It can even push you to decide _not_ to build your big idea, saving you valuable time and resources.

Another great benefit of an MVP approach is that it allows you to test a hypothesis with minimal resources. If you have no money in the bank, you can still [get something off the ground](http://venturebeat.com/2013/12/10/homeless-coder-prevails-over-skeptics-releases-mobile-app-to-get-off-the-streets/).

## History

In the startup world, the idea of MVP was popularized by [Steve Blank](http://www.amazon.com/Four-Steps-Epiphany-Steve-Blank/dp/0989200507) and [Eric Ries](http://www.startuplessonslearned.com/2008/10/about-author.html). Eric's 2008 blog post, [The Lean Startup](http://www.startuplessonslearned.com/2008/09/lean-startup.html), kicked off a movement in software development toward building companies around the idea of testing business hypotheses in an iterative way. The idea of MVP is central to this approach, and has become part of the _lingua franca_ of startup culture.

The origins of MVP (and lean software development) draw on the Toyota corporation's [lean manufacturing approach](http://en.wikipedia.org/wiki/Lean_manufacturing), called the _Toyota Production System_ (TPS). Toyota bigwig [Taiichi Ohno](http://en.wikipedia.org/wiki/Taiichi_Ohno) coined the idea of "Just In Time" production, in which return on investment is maximized by reducing inventory.

Among other things, the TPS introduced the idea of [Kanban](http://en.wikipedia.org/wiki/Kanban). The key takeaway of TPS as it applies to software is this: ***production is determined according to the actual demand of the customer***.

## It's Not What You Think It Is

We've all seen this picture right?

![](https://blog.engineyard.com/images/blog-images/actually-mvp.png)

Raise your hand if you think it's a good idea.

Now raise your hand if you think you actually follow it. Really? Are you sure you didn't motorize your skateboard? Put a steering wheel on your bike? Let me ask you this: did you actually do a customer interview at any point to see if they even _wanted_ a car?

An MVP may not be what you think it is. Eric Ries [defines it](http://www.startuplessonslearned.com/2009/08/minimum-viable-product-guide.html) as "that version of a new product which allows a team to collect the maximum amount of validated learning about customers with the least effort". What does that actually mean?

It means building just enough of a product to be deployed and used. It's the _minimum_ feature set that you need to find out whether it makes sense to invest further in an idea. If you're building a dating site for dogs, what do you need? Profiles and messaging. You don't need favoriting, automated emails, or the ability to see who viewed your profile.

If you're breaking ground with a new idea, you can ask yourself: would people use this?
If you're spinning something that's already out there, ask: would people love this more than what they're using now?

Then answer that question. Build profiles and messaging, put it in front of some users, and see if they love it. Track clicks, invite them in for an interview, get their email and reach out personally. Identify *[Key Performance Indicators](http://en.wikipedia.org/wiki/Performance_indicator)* (KPIs) and use them to verify success.


If they love it, keep going! Add a link to favoriting that doesn't work. Track clicks. If enough people are trying to favorite, build it. Invite users in for an interview. Get their email and reach out personally.

If you try several different feature sets and find out that nobody wants a dating site for dogs, it's ok. In fact, that's great. That means you really did it. You tested the market cheaply _before_ you sunk bags of money into building Doggie Dates. That's a win.

An MVP is not just doing sprints to build your product over time. It’s an _experimental process_.

It's a small vision, tested and validated thoroughly before moving forward.

It's not twelve weeks of development and a release.

It's continuous deployment.

It's [not a buggy alpha site](http://vincentjordan.com/2012/01/why-is-your-minimal-viable-product-mvp-really-just-a-pos/) with ten features that _kind of_ work.

It's the two most defining features needed for the product to be useful.

It's bare, not broken.

On the other hand, an MVP approach is [not just _release early, release often_](http://www.startuplessonslearned.com/2009/03/minimum-viable-product.html), either. Yes, build a small thing. Yes, gather feedback and incorporate it. But don't let the feedback cause you to pivot so hard that you can't remember what you were trying to do in the first place. If you find yourself testing Amazon for dog toys instead of a dating site for dogs, you haven’t pivoted for product-market fit, you’ve pivoted to an entirely new idea.

Start with a vision and stay true to it. Build the skeleton, and let feedback from early users help you flesh out the details.

## Really MVP

So you want to build a thing, huh? You're going to change the world, like Steve Jobs? Slow down there, buckaroo! I hope you know what you're getting into.

According to [this article](http://get2growth.com/how-many-startups/), there were 1.35 million tech startups as of February 2014. Before you start dreaming of all that VC cash and crack your wallet open to get things off the ground, why don't you do a little experiment to see if it's at all likely that you'll end up anywhere besides broke.

The lean startup approach is all about the [build-measure-learn](http://theleanstartup.com/principles) cycle. Before you have a company, you'll have to start by building something.

### First Step: Build

What's the smallest first step you can take?

The classic example of an MVP you can use to test an idea is a landing page with a sign up form. The idea is to build a pixel-perfect landing page touting all the benefits of joining Doggie Dates and deploy it on an easy-to-use platform like [Engine Yard](https://www.engineyard.com/). Then you drive users to the site by purchasing some Google AdWords, and entice them to sign up with their email for early access to the application.

There has been some recent debate as to whether a landing page is even an MVP. Although some would say no, like [Ramli John](http://ramlijohn.com/a-landing-page-is-not-a-minimum-viable-product/), this strategy doesn't provide enough insight to complete the build-measure-learn cycle. [Eric Reis](http://www.startuplessonslearned.com/2009/03/minimum-viable-product.html) and [others](https://medium.com/@joelgascoigne/how-to-successfully-validate-your-idea-with-a-landing-page-mvp-ef3c2d02dc51) seem to disagree. A landing page _can_ provide enough information to build a successive MVP and continue to gather feedback. If they sign up they're probably interested in what you're selling.

Although Ramli John isn't a fan of landing pages, he does have some other great suggestions for ways to build a first MVP. Some startups, like AngelList, have begun as an email list. Blogs can be another great place to gather interested users. We've talked about Eric Reis quite a bit already. His [blog](http://www.startuplessonslearned.com/2008_08_01_archive.html) covered topics like refactoring, TDD, and fundraising before he was known for the Lean Startup. You can try getting off the ground with a video and a startup campaign. Finally, you can do it the old fashioned way: _the hustle_. Just sell your service. In person. _Before_ you build an app.

Landing pages are great for founders of small startups and developers with a great idea for a side project, but what does MVP mean if you have another sort of job?

If you're a consultant like me, encourage your clients to prioritize their feature requests. We have so many awesome things in mind for Doggie Dating! Favoriting, profile walls, see who viewed me, a Dogs You Might Dig service, responsive layouts, native mobile versions... The list goes on and on. It's great to write them all down and put them in a tracking tool like [Sprint.ly](https://sprint.ly/), so that you don't lose all these creative ideas. Then it's time to prioritize.

What do you need first? What is the smallest version of your idea that people (or dogs) could possibly use? Put those tickets at the top of your tracker, and mark the point when they'll be done with a release bar. Encourage your clients to ask themselves: "is this necessary for people to use the site?" before you put _anything_ above that bar.

Once you've reached it, follow the steps below! Measure and learn before you go on. That way, you can collectively decide what should _actually_ be built, instead of spending the client's money building things that are going end up being discarded.

If you're a Product Owner, maybe you're charged with exploring new ways that your company can gain more users, or convince the existing ones to pay more. There are some cool tricks you can use to sneak a feature MVP into what's already there. You can make  links that claim to take the user to one or more features, but are actually inactive (preferably with a modal dialogue to explain what’s going on). Then you track the clicks and decide what to build from there. Or you can ask users to pay for a certain feature before you actually begin development on it. If you don't reach a certain threshold of sign-ups, you just cancel it, apologize, and refund the money.

No matter your role, if people don't seem to want what you've put out there, delete it and build another version of your dream. Be happy about all the time and money you just saved by not building something that nobody wants! Keep going until you find something that sticks. In this way, you'll set up a great foundation on which to build the rest of your business.

### Second Step: Measure

Regardless of how you choose to build the first version of your idea, you'll want to measure user engagement once it's deployed. The easiest way to do this is to add Google Analytics into the page and track clicks and pageviews. Reflecting on this information, you'll begin to learn what people want. If anyone actually signs up on your landing page with their email, you can reach out personally (maybe even take them to lunch) and ask them questions to learn more about what people would want to use.

Once you've gotten past the first iteration of your MVP, you can also invite customers to user testing sessions. These sessions can offer great insight into how your application should behave, and which features are misunderstood or unwanted. Finally, [A/B testing](http://www.startuplessonslearned.com/2008/09/one-line-split-test-or-how-to-ab-all.html) can be a great way to research which direction to go next once you've passed the early stages.

### Third Step: Learn

After you've measured clicks, user responses, user testing, and A/B testing results, you can begin to draw conclusions. Maybe dating dogs don't care about favoriting. Delete that feature. Maybe you heard over and over again that having a profile "wall" would make a huge difference to users. Perhaps that should be the next thing you build?

You need to sift through all of the information that you get and decide how to act.

## Conclusion

In the software industry, a lot of people pay lip service to the idea of a Minimum Viable Product. But for many of us, it's not what you think it is.

If you're thinking: I know what people want, and I'm going to build it, you've already misunderstood the process. MVPs are experiments, research. How you use them differs a little bit depending on your situation, but the basic premise is the same. Build a small thing, measure the way it's used, learn from it, repeat.
