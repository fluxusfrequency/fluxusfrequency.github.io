---
layout: post
title: "IdeaBox Retrospective"
date: 2013-10-26 06:42
---

This week was a long haul. We were focused on building [IdeaBox](http://tutorials.jumpstartlab.com/projects/idea_box.html), an idea catching app. It's kind of like [Evernote](http://www.evernote.com). Mine turned out pretty well. Using some strict time management, I was able to complete all of the extenstions listed on the assignment.

Here's a link to mine: [My IdeaBox](http://ideabox-flux.herokuapp.com/)

And a screenshot:

{%img http://s10.postimg.org/40ywl3vvt/Screen_Shot_2013_10_26_at_6_54_54_AM.png, 'IdeaBox', 'Screenshot of a web app' %}

## Scheduling My Workflow

Here are three things I discovered along the way:

1. Planning a project timeline is key.
2. I have a hard time with design.
3. It's difficult to write Capybara tests when you don't know the layout of your app.

This is the timeline I used to finish building the full app in a week. We were simultaneously learning how to build linked lists, write SQL statments, and use the Seqeul gem.

```
Wed: Setup Models, Tagging
Thu: Statistics, Search, Fuel
Fri: Revisions and Groups
Sat-Sun: Rest, catch up if needed
Mon: Users and Groups
Tue: Sound and Image
Wed: Front End, Mobile, and SMS
```

## Trouble With Design

Following this plan, I was able to get the app up and running by Thursday. I spent Wednesday waffling between css layouts - Zurb [Foundation](http://foundation.zurb.com/), [Semantic UI](http://semantic-ui.com/), and [HTML Kickstart](http://www.99lime.com/) - trying for the life of me to make it look decent. Despite being a creative person and having read half a dozen web design books in the last year, I was having a lot of trouble feeling satisfied with my look. You can see what I came up with by [visiting my IdeaBox](http://ideabox-flux.herokuapp.com/).

## Testing

I was surprised to find that I had a hard time practicing TDD for the views. Using Capybara early on, my tests were constantly blowing up when I got around to changing the layout. I don't really think it matters, though. I was able to test my models with unit and integration tests easily enough. Then I added the important routes to my Capybara acceptance test after the web views were built, so if I want to tweak things later, I will have test coverage.

## Takeaways

Here are some other parts of web development I learned from this project:

- Using Sintra to send files to users from a dynamic URL exposes your computer to the web.

```ruby What Not To Do
get '/download/:filename' do |filename|
  send_file "./files/#{filename}", :filename => filename, :type => 'Application/octet-stream'
end
```
- How to salt a password before encrypting.
- How to use the RACK_ENV for testing vs. deployment.
- How to use [Twilio](https://www.twilio.com/) to set up texting to my app.
- How to use [ngrok](https://ngrok.com/) to expose your local server to the web for testing. I needed this for Twilio.
- Don't try to switch from Yaml Store to Postgres with ActiveRecord when you only have 24 hours left before launch.
- When trying to build a lot of features in a short time, code quality can suffer. I'd like to refactor this at some point.

## If I Were to Start Over

At this point, if I could go back and start over, there are two things I'd do differently:

1. Use an SQL database from the start.
2. Design the view earlier on in the process, to facilitate a better testing process with Capybara.


In the end, I was pretty proud of my IdeaBox. I learned that I can actually build and deploy an entire web app on my own! Full-stack!-ish...

##Code Retreat

On Friday, we left Galvinize and went to work in a different space - The Source in Rino - for our first code retreat. We spent the day pairing with different people in the class on two exercises: programming "99 Bottles of Beer on the Wall", and a robot instruction simulator. I had a great time getting to work with a wide variety of my classmates, including some I had not yet paired with. My favorite constraint is when we could only one line of code before switching off, and without talking. I was also trying to practice some vim skills, which my pairs were gracious enough to help me with. One thing I really disliked was the space at The Source itself. It was too loud and filled with echoes. For me, a space that was calmer and quieter would have been preferable. Aside from that, though, I hope the next code retreat will be similar - a chance to work with many peers on fun and challenging problems.

