---
title: Full Stack Javascript
date: 2013-11-22 11:12:00 Z
categories:
- source
layout: post
---

## UPDATE:

I now have a working version of JetFuel Express live on Heroku at [http://jfx.herokuapp.com](http://jfx.herokuapp.com/). Please check it out!

This was JavaScript week at gSchool. For me, this was great. As I've been learning Ruby, I've been working on a lot of JavaScript on the side. From what I hear, knowing JS is going to be indispensible as I enter the professional development world. Having done a bit of browser scripting with vanilla JS and jQuery in the past, I've recently been digging into [libraries and frameworks](http://fluxusfrequency.github.io/blog/2013/10/26/so-many-javascript-libraries/). In the last couple of weeks, I've been focusing really hard on Backbone.js in particular, and this week gave me a chance to apply my skills with it to build a project. As I went along, I discovered quite a few other new tools written in JavaScript. After hacking away all week, my key takeaway was this:

You can develop full stack with Javascript.

I decided to try to build [JetFuel](http://tutorials.jumpstartlab.com/projects/jet_fuel.html), a Jumpstart Lab project in which you clone [Bitly](https://bitly.com/). But instead of building it with ActiveRecord and Sinatra, I decided to build it all in JS.

At the moment, I have a basic link shortening service working. Here's a glimpse of what it looks like:

{% img http://s5.postimg.org/ihxudv1xj/Jet_Fuel_Express_Production_Shot.png 'JetFuel Express' 'A Bitly clone website in production' %}

And here are the technologies I'm using:

## The App

### [Backbone.js](http://backbonejs.org/)

I've been working really hard at learning Backbone, a client-side MVC (actually MV*) library, over the past few weeks. I had started to look into Ember.js, and was having a hard time. My mentor, Mike Pack (who works at [Mode Set](http://modeset.com/)), suggested looking into Backbone first, because: "you're going to burn a lot of mental cycles trying to learn Ember and Rails at the same time". This has turned out to be some really good advice. For me, Backbone fairly intuitive, yet still quite challenging to learn.

I did a few tutorials. I found this [RailsCasts](http://railscasts.com/episodes/323-backbone-on-rails-part-1) one to be the most approachable. This [Nuttuts+ Premium Tutorial](https://tutsplus.com/course/backbone-on-rails/) was interesting. It was written with Coffeescript and Handlebars. There were [some](http://coenraets.org/blog/2011/12/backbone-js-wine-cellar-tutorial-part-1-getting-started/) [others](https://www.youtube.com/watch?v=FZSjvWtUxYk) that were less remarkable, but they still helped me get familiar with the library.

I still felt like I needed to understand more, so I checked out the book [Developing Backbone.js Applications](http://addyosmani.github.io/backbone-fundamentals/) by Addy Osmani, available free online. It covers many Backbone topics, and has quite a few tutorials. In the first one, it walks you through building a simple to-do app called [TodoMVC](http://todomvc.com/) in Backbone.

My favorite tutorial from the book was the [Book Library](http://addyosmani.github.io/backbone-fundamentals/#exercise-2-book-library---your-first-restful-backbone.js-app) tutorial. It really makes it clear how to use Backbone to talk to an API. It also teaches you build that API in Express.js. It was essentially the inspiration for my app.

Backbone can be really hard to debug. The value of the keyword 'this' can get kind of tricky, in particular. For example, in my UrlsView class, the value of 'this' shifts from the Backbone View object to the urlCollection, so I have to assign it to 'that' before passing it into the callback function. This was a subtle problem that was breaking my view, until Mike helped me fix it.

```javascript

var jetfuelexpress = jetfuelexpress || {};

jetfuelexpress.UrlsView = Backbone.View.extend({
  el: '.url-list',

  initialize: function(initialUrls) {
    this.urlCollection = new jetfuelexpress.UrlCollection();
    var that = this;
    this.urlCollection.fetch({reset: true, success: function() {
      that.fetched = true;
      }
    });
    this.render();

    this.listenTo(this.urlCollection, 'add', this.renderUrl);
    this.listenTo(this.urlCollection, 'reset', this.render);
  }

});

```

Here's a look at how I built the rest of my stack:


## The Database

### [MongoDB](http://www.mongodb.org/)

I'm using of MongoDB to store my data. It plays nicely with my other components, since it stores documents in a JSON-like format. It's also handy that it uses JavaScript internally on the server-side, and has a shell that's based on JS. As a database, it feels a little crazy, because it doesn't auto=increment an id attribute like Postgres. I like it though. It's fast and flexible. My server interfaces with the database via:

### [Mongoose.js](http://mongoosejs.com/)

It's summed up well on the Mongoose website. Mongoose is a "MongoDB object modeling tool designed to work in an asynchronous environment". It's like DataMapper or ActiveRecord in Ruby. It has some really powerful validations baked in:

```javascript

var UrlSchema = new mongoose.Schema({
  originalUrl : { type: String, required: true, validate : [
            function(u) { return u.match(/^((http(?:s)?\:\/\/)?[a-zA-Z0-9\-]+(?:\.[a-zA-Z0-9\-]+)*\.[a-zA-Z]{2,6}(?:\/?|(?:\/[\w\-]+)*)(?:\/?|\/\w+\.[a-zA-Z]{2,4}(?:\?[\w]+\=[\w\-]+)?)?(?:\&[\w]+\=[\w\-]+)*)$/); },
            'Visits must be positive!'] },
  slug        : { type: String, required: true },
  active      : { type: Boolean, required: true },
  visits      : { type: Number, required: true, validate : [
            function(v) { return v >= 0; },
            'Visits must be positive!'] },
  userId      : { type: String, required: true },
  createdDate : { type: Date, required: true }
});

```

## The Server

### [Node.js](http://nodejs.org/)

Built on Google's V8 JavaScript engine, Node is a super fast platform that compiles JS directly to the hardware without having to be interpreted. It has "a built-in HTTP server library, making it possible to run a web server without the use of external software".

### [Express.JS](http://expressjs.com/‎)

This lovely framework is the foundation of my backend. It is architecturally based on Sinatra, and is very similar to work with. It was really easy to get a project up and running quickly in Express using Node Package Manager (see below). Since it's built on Node, it is very fast. I used it to set up an API server to send and recieve JSON requests from my front end.

## The Tests

### [Jasmine-Node](http://pivotal.github.io/jasmine/)

We used this testing framework quite a bit in class for [Exercism](http://exercism.io/) exercises. It felt like a natural choice. Although I am usually more comfortable with 'assert' style test statements like you find in Minitest, I found Jasmine to be pretty intuitive. Here's an example of one of my tests:

```javascript

describe("api route", function() {

  beforeEach(function() {
    Url.remove({}, function(){});
  });

  it("should respond successfully with to an API request", function(done) {
    request("http://localhost:3000/api", function(error, response, body) {
      expect(response.statusCode).toBe(200);
      expect(response.body).toContain("JetFuelExpress API is running!")
      expect(error).toBe(null);
      done();
    });
  });
});

```

### [Mocha](http://visionmedia.github.io/mocha/)

A couple days into this project, I was tweeting about what I was up to, and an Express.js dev tweeted back at me, suggesting I check out his [Express.js tutorials](http://webapplog.com/expressworks/). He also recommended the Mocha testing framework. I haven't had a chance to dig in yet, but it looks promising. The website says that you can use your favorite style of assertion ('should', 'expect', or 'assert') by loading different libraries - Chai, Should.js, Expect.js, or Better Assert.

### [Zombie.js](http://zombie.labnotes.org/)

I was looking around for some kind of acceptance test solution along the lines of Capybara. I couldn't really find anything I liked, but I did stumble on this "headless browser" testing framework. Although it looks promising, I've had a lot of trouble making it work the way I want. I wouldn't really recommend it, unless you have extra time to figure out how to make it do what you want.


## The View

### [Handlebars.js](http://handlebarsjs.com/)

After giving a lightning talk about [Mustache](http://mustache.github.io/) a few weeks ago, I just had to try Handlebars for this project. It went pretty well, as long as I didn't try to inject anything that was null into my templates. When I did that, it killed the server.

### [Bootstrap](http://www.getbootstrap.com)

Because I was already in over my head with all of these JavaScript libraries, I decided to punt on the design for this one. I bought a template from [Wrap Bootstrap](http://wrapbootstrap.com/).

## Bits and Pieces

### [Grunt.js](http://gruntjs.com/)

Having a browser fetching 18 JavaScript files can be slow, so it's nice to uglify and concatenate them all into one big file. Grunt can automate this task for you. It's sort of like Rake in Ruby. Here's what that looked like. I copied the code from gSchool[0] student Raphael Weiner's [Elefeely UI](https://github.com/raphweiner/elefeely-ui).

```javascript

module.exports = function (grunt) {
  grunt.initConfig({
    connect: {
      server: {
        options: { port: "3000",
                   hostname: "localhost",
                   keepalive: true }
      }
    },
    uglify: {
      build: {
        files: {
          'main.js': ['main.js']
        }
      }
    },
    concat: {
      files: {
        'src': ["public/javascripts/lib/jquery-1.10.1.min.js",
                "public/javascripts/lib/json3.min.js",
                "public/javascripts/lib/handlebars.js",
                "public/javascripts/lib/underscore-min.js",
                "public/javascripts/lib/backbone-min.js",
                "public/javascripts/app.js",
                "public/javascripts/routers/router.js",
                "public/javascripts/models/url_model.js",
                "public/javascripts/collections/url_collection.js",
                "public/javascripts/views/app_view.js",
                "public/javascripts/views/header_view.js",
                "public/javascripts/views/footer_view.js",
                "public/javascripts/views/shorten_view.js",
                "public/javascripts/views/home_view.js",
                "public/javascripts/views/url_view.js",
                "public/javascripts/views/urls_view.js"],
        'dest': 'main.js',
      }
    }
  });
  grunt.loadNpmTasks('grunt-contrib-connect');
  grunt.loadNpmTasks('grunt-contrib-concat');
  grunt.loadNpmTasks('grunt-contrib-uglify');
  grunt.registerTask('build', ['concat', 'uglify']);
  grunt.registerTask('default', ['build']);
};


```

### Node Package Manager (NPM)

I really like how easy it is to install Node.js dependencies. I've been spoiled with Bundle, so it felt comfortable to be able to add things I needed with ease. You just declare your dependencies in a package.json file in your application root, then run 'npm install'. The package file is in JSON, so it's not as pretty as a Gemfile, but it will give you all the things you need. Here's what mine looks like:

``` javascript

{
  "name": "JetFuelExpress",
  "version": "0.0.1",
  "description": "A url shortening service",
  "private": true,
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "3.4.4",
    "jade": "*",
    "less-middleware": "*",
    "path": "~0.4.9",
    "request": "2.27.0",
    "mongoose": "~3.5.5",
    "grunt": "*",
    "grunt-contrib-connect": "*",
    "grunt-contrib-concat": "*",
    "grunt-contrib-uglify": "*"
  }
}

```

## Wrapping Up

Coming to JS app development from Ruby, I felt really uncomfortable at first. There are ***so*** many JavaScript libraries and frameworks. I was pretty sure I was in over my head. But somehow I held it together. I think it's because JS has components that serve the same functions as certain Ruby gems:

Mongoose is kind of like ActiveRecord.

Express  is kind of like Sinatra.

Jasmine  is kind of like RSpec.

Zombie   is kind of like Capybara.

Grunt    is kind of like Rake.

NPM      is kind of like Bundle.

Backbone isn't really like anything, but at least its MV* pattern is close enough to Rails that I feel at home.

I saw this [cool article](http://coding.smashingmagazine.com/2013/11/21/introduction-to-full-stack-javascript/) in the JavaScript Weekly newsletter today. Reading it made me understand what I'd been learning all week: full stack JavaScript development. Up until then, I hadn't quite seen it for what it was.

JS is not as friendly an environment as Ruby, where you get to use intuitive syntax, the community is better established, and there are lots and lots of gems available. But it is very fast and scalable, and at least the components feel familiar when transferring from Sinatra and Rails.

I'm looking foward to continuing with JetFuel Express. It's been a good project for extending my understanding of JavaScript.
