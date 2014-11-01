---
layout: post
title: "Dinner Dash Complete"
date: 2013-11-18 08:12
---

Last Thursday, my group and I wrapped up the [Dinner Dash](http://tutorials.jumpstartlab.com/projects/dinner_dash.html) tutorial we were working on. It was essentially the website for a restaurant with an administrative back end.

Here's a link to our [Dinner Dash Project](http://onoburrito.herokuapp.com/).

{%img http://s5.postimg.org/5ddkt77uv/Ono_Burrito_Website_Screenshot.png 'Ono Burrito' 'A Ruby on Rails website' %}

I learned a lot. Aside from this being my first full-blown Rails project, I discovered many other useful things about Rails projects along the way.

## Rails Gems

Here are some of the technologies I tried out for the first time

- [SASS/CSS](http://sass-lang.com/). I intentionally spent a lot of time on the front end in this project. Although I feel strong in HTML5/CSS3, I would really like to get better at design. Because of this, I worked hard on building up the view ***without Bootstrap***, using partials and SCSS to make things cleaner. I was also reading [The Rails View](http://www.therailsview.com/), and tried to incorporate lessons learned from it into what I was doing.

- [The Rails Guides](http://guides.rubyonrails.org/). After losing my mind trying to figure out how to work with form_for during the first week of the project, I discovered the Rails Guides. I read all of the ActiveRecord and ActionView documentation over the weekend, and felt a lot more confident afterwards. The Rails Guides rock!

- [Active Admin](http://www.activeadmin.info/). This little number let us add an administrative backend to our app with next to no effort. It was kind of cheating to use it here, but it was also really cool to find out about. It may come it very handy in the future.

- [Jazz Hands](https://github.com/nixme/jazz_hands), a gem that improves the Rails Console by making it Pry-based, and adding pretty print to make it easier to look through data.

- [Simplecov](https://github.com/colszowka/simplecov). This is a great way to check your test coverage. I'm somewhat ashamed to admit that our project was only about 69% covered.

- [Faker](https://github.com/stympy/faker). Made it easy to generate seed data for our database.

- [Quiet Assets](https://github.com/evrone/quiet_assets). I got pretty tired of looking at the assets in Guard every time I ran a test. Quiet Assets made the server logs a lot easier to look at.

## Soft Skills

One of the most challenging aspects of the project for me was managing the social aspect of this project. Since I was percieved as the strongest member of our group programming-wise, the others looked to me for guidance. But I am not a natural leader. Although I was able to chart our best course for finishing the project, I didn't do a great job of checking in with everyone emotionally. At the end, I found out that one member of my group felt left out and behind. In the future, when I lead, I will work to make more space for people to express how things are going, not just in terms of code progress, but also how they're feeling about the project.

We also used [Pivotal Tracker](https://www.pivotaltracker.com/). This was a first for me. Although I liked learning to use Gherkin, I wasn't wild about the application's UI. We ended up switching to Milestones and Issues on Github for the last part of the project.

However, looking back now, I actually think that using Pivotal Tracker did improve my experience, compared with the previous project. I'll definitely give it another go for the next project. I would really like to try translating the Tracker stories into Capybara or Cucumber tests right out of the gate.

## What's Next

This week, we'll be working on JavaScript. I'm really excited about this, because I've been working hard at it on the side. Right now I'm working through Addy Osmani's [Developing Backbone.js Applications](http://addyosmani.github.io/backbone-fundamentals/), and I'm loving it. I'm also building a Backbone app with a Rails API with my mentor as a side project.

After Thanksgiving, we'll be building a multi-tenant platform based on Dinner Dash: [Fourth Meal](http://tutorials.jumpstartlab.com/projects/fourth_meal.html). It looks intimidating, but I'm stoked to be able to build a platform like this. It seems like you could build a great business around a platform like this. [Bandzoogle](http://bandzoogle.com/) comes to mind.

We're less than halfway through gSchool now, but we've learned many of the skills we'll need as Ruby on Rails developers. I'm glad that we'll have time to practice these skills for the rest of the program. I think I'll feel very confident about what I'm doing when we graduate in February. I shudder to think that if I had gone to a different bootcamp, I might be heading out into the job market right now. Although I can certainly build web apps now, I still have so much to learn. I'm really glad to be at gSchool; the curriculum is excellent, my peers are supportive, hardworking, and smart, and my teachers are always willing to go the extra mile to help me figure things out.
