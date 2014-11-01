---
layout: post
title: "Getting Started With Rails"
date: 2013-11-08 10:15
---

This week at gSchool, we're digging into our first major Rails project, [Dinner Dash](http://tutorials.jumpstartlab.com/projects/dinner_dash.html). It's been a challenging transition.

## Rails Challenges

Although I know some Ruby, and I know how to drive development with tests, using Rails is a whole new ball game. There are so many new words! From ActiveRecord associations, to "form_for" and "button_to", to strong params, there's a huge library of methods I don't know.

One particularly difficult challenge for me this week was figuring out how to properly use "form_for". I was having a little trouble with forms and HTTP routes. I know how to write an update form like this in HTML:

```html
<form action="/ideas" method='POST'>
  <input type='text' name='idea[title]' placeholder="Title" value="title"><input/>
  <input type="hidden" name="_method" value="PUT"><input/>
  <input type='submit' value='Submit'><input/>
</form>

```

But I was having a lot of trouble submitting PUT and DELETE verbs with "form_for" in Rails.

I was trying to create a form with some code like this, but it wasn't really working out:

```erb
<%= form_for :idea, (action: :put)?? do |f| %>
  <p>
    <%= f.label :title %><br>
    <%= f.text_field :title %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>

```

Franklin eventually straigtened us out with a different approach, which he described as the "Rails-y way" to do it:

```erb
<%= link_to "Title", ideas_path, method: :put %>
```

I assume there will be many more "Rails-y" idioms to learn in my near future.

## Organizing Our Workgroup

My group for this project is comprised of four people: myself, Bree, Nikhil, and Tyler. So far, we've been working really well together. We're using [Pivotal Tracker](https://www.pivotaltracker.com) and [Campfire](https://campfirenow.com/) (this is my first time for both) to drive our development and communication. I've found it hard to get used to Pivotal Tracker. Although I enjoyed learning how to write epics and stories in [Gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin), I'm used to thinking through problems one step at a time in coding. Having to go back to the website and refer to what's in the 'Current' or 'Icebox' list interrupts my flow.

I'm also not yet at the point of driving everything with [Capybara](https://github.com/jnicklas/capybara). Although we're testing our models before we implement them, we aren't doing any integration tests in advance. Maybe if we were to transform our Gherkin stories into Cucumber, it would be an easy transition. I'm a little shy, though, since it always takes a while to get to know a new testing framework.

Here's one testing success, though: we've integrated [Guard](https://github.com/guard/guard) into our workflow. This is the first time any of us have used it successfully.

I'm also trying to itegrate some of the lessons learned from [The Rails View](http://www.therailsview.com/), which I'm currently reading. In this project, I'm trying to take on a lot of responsibility for the view layer, because I don't feel strong in design and would like to get better. Here's our current view, four days in. Unfortunately, it's mostly Bootstrap. I'm planning to shine it up a bit this weekend.

{% img http://s5.postimg.org/8g4kan24n/Ono_Burrito_Beta_Site.png 'Ono Burrito' 'Screenshot of a website in development' %}

## Coming Back to Rails

Before gSchool, I did a few Rails tutorials online. Coming back to it now, after all the work we've done with Ruby, hand-built model associations, and SQL, I feel very positive. I may not know what I'm doing yet, but it's a lot easier to figure things out, now that I have a context for them. Categorizing confusing bits of code into categories comes much more quickly now. SQL, database tables, models, controllers, views, templates, Ruby expressions, Rails helpers...these ideas have a lot more meaning than they did before. This means I'm able to bounce back from confusion much more quickly.

## Moving Forward

I'm excited to get up to speed in Rails. Our group was able to build out a complete restauarant site, with basic users and ordering functionality, all in the course of a few days. No wonder they say it's easy to get up and running quickly with Rails!

I've also been working with my mentor (the most awesome Mike Pack) on building a basic API in Rails, as a service for a Backbone.js app on the front end. It will eventually be an online recipe book, with Facebook and Instagram integration. I'm really excited to continue working on it.

## Wrapping Up

I'm feeling a little bit intimidated by Rails at the moment, because there's so much to learn. But I'm not worried: I relish the challenge of learning how to use a new tool, which is one of the reasons I got into programming in the first place. Plus, there are so many quality resources out there for learning Rails, I'm sure I'll be up to speed in no time.
