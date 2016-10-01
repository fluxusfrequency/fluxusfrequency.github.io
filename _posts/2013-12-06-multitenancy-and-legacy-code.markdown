---
title: Multitenancy and Legacy Code
date: 2013-12-06 09:03:00 Z
categories:
- source
layout: post
---

This week at gSchool, we started a new rails project called [Fourth Meal](http://tutorials.jumpstartlab.com/projects/fourth_meal.html). The assignment's basic premise is to build a multi-tenant platform for restaurant ordering, along the lines of [GrubHub](https://www.grubhub.com/). This is our first time dealing with multi-tenancy. It's also the first time that we are building from a legacy code base - we're using our last projects. Mine was [OnoBurrito](http://onoburrito.herokuapp.com), and that's the project my group chose to build on for this assignment. Finally, this will also be the first time we optimize our apps for performance.

## Project Management

We're using Pivotal Tracker very seriously in Fourth Meal, with our instructors pretending to be clients that have hired us to build this app. I've found Tracker restrictive in the past. But I think I felt that way because I was using it as more of a to-do list than a project management tool. After having Jeff help us write about 13 clear stories yesterday, I feel a lot more positive about using it to drive my group's progress. At this point, Tracker is helping me feel more organized. I can translate stories into tests and use Capybara to drive development "from the top down".


## Legacy Code

Although we're working from code that I had a large part in writing, I feel like I'm floundering to turn OnoBurrito into a platform. There are a lot of things I would do differently if I were to build the app over again. For example, the category views are stuck inside of the item views, making it really hard to separate them. We also used Active Admin before, so we've had to rip it out and start the admin panel over again. My group decided to retrofit the admin part of Billy's restaurant onto Ono Burrito, and it's been kind of tricky to get everything working. Currently, I'm having a lot of trouble getting carts and orders working. It's hard to decide which of the pieces to store in the session, application, and database.

## Class Progress

We're now just past the halfway mark for gSchool. Although I can see how other programs would stop here and send their students out into the marketplace, I'm very glad that we have some time left to practice proper Test Driven Development, learn about scaling and performance optimization, and hone our Rails chops.

I do feel like I'm starting to lose momentum a little bit. I was going full-bore every day (except weekends) up until now, but I could feel that burnout was around the corner. So I've throttled back a little bit, taking time to cook, read, and exercise so I feel more balanced. I'm still totally dedicated to being the best developer I can be by the end of the program, and I think that these changes will help me do that. Meanwhile, I feel like a lot of my classmates have been making amazing progress. No matter who I work with lately, I feel like I am collaborating with a confident, smart, and capable programmer.

## Young Entrepreneurs

Last night we kicked off a project with Young Entrepreneurs. Several school-aged children came to Galvanize and met with us to get help building a website for their businesses. I'm in a small group with Nathaniel and Meeka, two classmates that I really enjoy working with. We're helping a 15-year-old entrepreneur named Jonah, who runs a local snack company. It's a little disheartening that we won't be able to use Sinatra or Rails to help him him, because he'll need the ability to update his site, and we likely don't have time to build a content management system. We'll probably help him get going on [Wix](http://www.wix.com), or a similar platform.

I feel a little conficted about this project. In a way, the kickoff was really bad timing, because I'm feeling so overwhelmed by Fourth Meal. On the other hand, it's nice to get back to working with kids. That's the thing I miss most about my previous career as a teacher. More importantly to me, I've been focusing on trying to find ways to be more generous, so this is a good opportunity to practice doing that.

## New Mentor

I met my new mentor, Justin Smestad, this week. He's a maintainer of the [Warden](https://github.com/hassox/warden) gem, a security specialist, and a musician. I think it's going to be a good fit. I'll miss my old mentor, Mike Pack, but I think that he's willing to keep working with me too. Now I'll have twice the support!

## Overall

I'm feeling really excited about web development. Thanksgiving break really helped me rest up and feel ready to tackle more. I am learning a lot of new things all the time, and I love it. On the other hand, I am feeling somewhat overwhelmed by Rails, and beginning to consider the job search makes me a little anxious. In the end, I'm optimistic. I think that the coming weekend will help me get grounded, and I'll come back next week ready to get some good work done on Fourth Meal.
