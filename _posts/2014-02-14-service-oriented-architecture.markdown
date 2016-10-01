---
title: Service Oriented Architecture
date: 2014-02-14 06:40:00 Z
categories:
- source
layout: post
---

We just finished up our penultimate project at gSchool, an application built using a Service Oriented Architecture (SOA). The project ideas were generated in small groups, and focused a the theme of health and wellness. Once the ideas were decided, our teacher Jeff Casimir generated a randomized list of the students, and we drafted projects.

I went first, and selected an idea that was close to my heart: an app to help home vegetable gardeners plan out their beds for the season. Having spent several years as an organic farm worker, garden company owner, and home garden hobbyist, I was excited to use this app. Every spring, before the ground warms up, my partner and I sit down with a pencel and graph paper to plan out the different beds in my garden: what will go where, how it will be spaced, when it will be harvested, and what we'll plant there afterwards.

I had big hopes for our project, dubbed [Planting Season](http://plantingseason.tk) ([code](https://github.com/VirginSoil)). A user would have several gardens, each with many beds. The app would keep track of her plans from month to month, and she would be able to click on a month to see what she'd planned to plant there, which spots were empty, and which plants were ready for harvest. It would find her USDA Hardiness Zone and suggest plants well-suited to grow there. If there was impending frost, or a bed needed water, it would send her a text message, email, or a Google voice message.

Ah, the dreams of the young and inexperienced...


## Planting Season's Structure

Two of my group members had just completed [FooFoBerry](https://github.com/foofoberry), an SOA app that exposed and consumed APIs from various project management tools, such as GitHub, Travis, and Code Climate. With their experience setting up services, I figured that getting the project up and running would be a breeze. In practice, it was more challenging than I expected.

I'm a big fan of a "lean startup" / "minimum viable product" approach in my projects. I like to get a simple product up as fast as possible, then expand on it, adding value. Given that we had three weeks, we planned three iterations for Planting Season:

Week 1 - Put up a landing page where users could enter their email to be notified when the app was ready.

Week 2 - Change the landing page to allow users to sign up or sign in, then send them to a dashboard page, where they could add and remove plants from a single garden.

Week 3 - Add functionality for multiple beds, text and email weather notifications, and a timeline for the garden through the year.

With this functionality in mind, we decided to build the following services:

1. Landing Page App
2. User Authentication App
3. Client-Facing Dashboard ("Coordinator") App
4. RESTful JSON API with Bed and Plant data
5. A gem to facilitate communication between the Dashboard and the API
6. External Services - Weather and Geolocation APIs

Here's how the project ended up looking in the end:

<br />

{% img center http://s5.postimg.org/w893s8cuf/planting_season_graph.png 'Planting Season Project Structure' 'A web application diagram' %}


## Project Management & Workflow

Over the course of gSchool, I've been learning a lot about the social aspect programming. I worked solo in my ealiest projects, then graduated to pairing, and later went on to work in groups of three or four. In the beginning, I was just doing my own thing, but as I started getting into the larger groups, project management became a key skill. I've found myself in the role of Project Manager in many successive projects. The first couple of times were messy: I didn't make enough space for people to express how they were *feeling* at our standups, our iteration planning was nonexistent, and I didn't understand how to use Pivotal Tracker at all. As I learned from these experiences, I began to firmly value things like daily standups with technical *and* emotional checkins, persona and wireframing UX design, iteration planning, proper use of Pivotal Tracker, consistent version control practices, and continuous integration.

We started Planting Season with the best of intentions, but this time around, it proved harder to practice these strategies than I hoped. At our checkins, it was clear that we were all feeling unfocused and pulled in different directions by job applications and interviews, burnout, and Jeff's absence after his wife gave birth to their latest addition to the family. We planned our iterations, but we didn't meet the first one because a couple of us were out of town or at interviews. Soon we fell behind. On top of that, Pivotal Tracker stopped letting us edit our stories, because our free trial was expired. The Travis CI specs wouldn't pass, because the specs couldn't talk to the API (more on this in a minute). Of all the agile practices we hoped to use, only the UX planning went well.

## Nginx, Passenger, and VPS Woes

Our troubles were compounded by the difficulty of getting our services to talk to each other. Our plan was to run Foreman locally, booting our apps onto different ports on localhost, then use Nginx to route everything through different namespaced routes on port 8080. Getting Foreman up and running was easy enough, but Nginx was a bear to configure. Meanwhile, getting Phusion Passenger and Nginx to play nicely on our VPS was also a struggle. It turns out that you have to install Passenger first, then add Nginx as an extension to it. If you install Nginx first, you end up with two conflicting versions. Additionally, one of our team mates was having RVM issues, and had to reinstall Ruby and all of his gems twice.

In the second week, we lost two solid days of of development time to the fight with the Nginx config file, Passenger, and RVM.

For posterity's sake, here's a snapshot of the `nginx.conf` file that finally worked for local production:

```bash

worker_processes  1;

error_log  logs/error.log;
#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    #passenger_root /usr/local/opt/passenger/libexec/lib/phusion_passenger/locations.ini;
    #passenger_ruby /usr/bin/ruby;

    server {
        listen       8080;
        server_name  localhost;

        # PLANTING SEASON

        # Landing Page
        location / {
          proxy_pass        http://127.0.0.1:3000;
          proxy_set_header  X-Real-IP  $remote_addr;
        }

        # Auth
        location /auth {
          proxy_pass        http://127.0.0.1:3001;
          proxy_set_header  X-Real-IP  $remote_addr;
        }

        # Coordinator
        location /dashboard {
          proxy_pass        http://127.0.0.1:3002;
          proxy_set_header  X-Real-IP  $remote_addr;
        }

        # API
        location /api {
          proxy_pass        http://127.0.0.1:3003;
          proxy_set_header  X-Real-IP  $remote_addr;
        }
   }
}

```


## Testing Troubles

Once we had the services running on a single port, we figured everything would be gravy. We added the VCR gem to save the results of our API calls, and began testing. We covered the landing page, auth app, and our gem.

Then I started to dig in on some Capybara tests for the coordinator app, and had some major headaches. In order to view the dashboard, I needed to set a signed cookie to simulate the authorization app. Easy enough. All you have to do is use:

```ruby

page.driver.browser.set_cookie 'user_id=1'

```

That is, unless you're using Poltergeist for a JS driver. Then, you clearly use:

```ruby

page.driver.set_cookie("user_id", 1)

```

But now, I had a different problem:

```bash

Failure/Error: within '#bed-functions' do
     Capybara::ElementNotFound:
       Unable to find css "#bed-functions"

```

Using the launchy gem and `save_and_open_page`, `#bed-functions` was there, all right. I could feel another configuration battle coming on as we headed into the third week.

## Getting Back on the Horse

On Monday morning of the week the project was due, we had little to show for our two weeks of work. There were services, but they couldn't talk to each other on the server. There were mock-ups, but the CSS wasn't in the views yet. The tests were thin. We had a meeting with Jeff, and told him we planned to bolster the test coverage and solidify what we already had. His advice: "that would be  a good idea if you had a thing, but right now you don't. You might want to make a thing first."

So we saddled up, and the cowboy coding began.

## jQuery Sparkle

The next two days were actually pretty fun, if a little painful (coding without tests makes me uncomfortable). We got the CSS hooked up, integrated with the weather service, and added a healthy dose of JavaScript to the dashboard. Although we would have liked to explore Ember.js, we decided to stick to plain old jQuery in the interest of time. We quickly coded 275 lines of jQuery sparkle, and we had a thing! If you're interested, go [check out](http://plantingseason.tk/) what we made!

My favorite part was adding the ability to click and drag on the garden bed squares for multi-selection. Here's what that looks like:

```javascript

$(function() {
  window.isMouseDown = false;

  $('td').mousedown(function(e){
    window.isMouseDown = true;
    var element = $(e.currentTarget);
    var thisClass = $(this).attr("class");
    toggleSquare(thisClass, element);
    return false;
  });

   $('td').mouseenter(function(e){
    if (window.isMouseDown){
      var element = $(e.currentTarget);
      var thisClass = $(this).attr("class");
      toggleSquare(thisClass, element);
    }
    return false;
  });

  $(document).mouseup(function () {
    window.isMouseDown = false;
  });

});

```

If you want to see more of what we did with the JS, you can check out the [code on GitHub](https://github.com/VirginSoil/planting_season_coordinator/tree/master/app/assets/javascripts).

## Conclusion

From the beginning of gSchool, the SOA project was one of the things I was looking forward to the most. Although Jeff said he thought that it was probably "not a reasonable thing" to start a project from the beginning with SOA, I set out with the highest of expectations. But the problems along the way were numerous.

The job search was a distraction. But more troubling to me was the difficulty of testing. I've gotten really comfortable with using Travis CI, and driving my development with Pivotal Tracker user stories translated into Capybara feature tests. It was really hard to do with services. Between setting up Passenger and Nginx and the testing troubles, we lost a *lot* of time on configuration.

Reflecting on the experience, I'd say that SOA is probably a practice better left until it is needed. If I were in charge building a new app for a startup, I would still start by iterate on delivering business value with successive MVPs. When I began to feel the pain of scaling, then I would think about splitting off services.

However, it's possible that I've only come to this conclusion because the process was so painful. There's another way of looking at this.

Here's a lesson that I've learned about programming again and again over the last six months:

```
The first time you do something it is hard and doesn't make any sense.
The second time you do it, it seems vaguely familiar, and makes a little bit of sense.
From then on, it makes sense and feels natural to do it.
```

If this is true (which it seems to be for me), then it might actually make sense to build apps using services from the get go, if you expect them to have to scale down the road. I'd love to nail down a workflow for getting Passenger, Foreman, and Nginx set up in less than 30 minutes. I'd also like to learn how to better test services and get them set up with continuous integration. I think that if I understood these things, SOA would be a tool I'd be a lot more likely to reach for.

If you have any tips about these two aspects of services, let's talk! Tweet at me: @fluxusfrequency.
