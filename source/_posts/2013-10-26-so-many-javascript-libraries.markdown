---
layout: post
title: "So Many Javascript Libraries!"
date: 2013-10-26 07:11
---

I've gotten pretty interested in client-side web applications recently. I've been looking into [Ember.js](http://www.emberjs.com). I find it to be an exciting framework, because it allows a scalable, MVC model app to be run with data storage happening on the client side. That, and it's being developed by Yehuda Katz, who I greatly admire.

After digging into the TodoMVC tutorial in Ember, I got to thinking more about Javascript. I then realized that there are many, many Javascript libraries out there, and I had no idea what was what. So I spent an evening scouring the web for information on them.

Here's a list of popular libraries that I compiled, with descriptions from their official sites and/or wikipedia.
<br />

#Javascript Libraries

##Ember.js
An open-source client-side JavaScript web application framework based on the model-view-controller (MVC) software architectural pattern. It allows developers to create scalable single-page applications by incorporating common idioms and best practices into a framework that provides a rich object model, declarative two-way data binding, computed properties, automatically-updating templates powered by Handlebars.js, and a router for managing application state. Most Ember.js applications also use Ember Data, a database-independent data persistence library that provides object-relational mapping (ORM) facilities for the web.

##Angular.js
An open-source JavaScript framework, maintained by Google, that assists with running single-page applications. Its goal is to augment browser-based applications with model–view–controller (MVC) capability, in an effort to make both development and testing easier.
The library reads in HTML that contains additional custom tag attributes; it then obeys the directives in those custom attributes, and binds input or output parts of the page to a model represented by standard JavaScript variables. The values of those JavaScript variables can be manually set, or retrieved from static or dynamic JSON resources.

##Backbone.js
Backbone.js gives structure to web applications by providing models with key-value binding and custom events, collections with a rich API of enumerable functions, views with declarative event handling, and connects it all to your existing API over a RESTful JSON interface.

##Marionette
Backbone.Marionette is a composite application library for Backbone.js that aims to simplify the construction of large scale JavaScript applications. It is a collection of common design and implementation patterns found in the applications that I (Derick Bailey) have been building with Backbone, and includes various pieces inspired by composite application architectures, such as Microsoft's "Prism" framework.



##Jasmine
An open source testing framework for JavaScript. It aims to run on any JavaScript-enabled platform, to not intrude on the application nor the IDE, and to have easy-to-read syntax. It is heavily influenced by other unit testing frameworks, such as ScrewUnit, JSSpec, JSpec, and RSpec.

##Sinon.js
Standalone test spies, stubs and mocks for JavaScript.
No dependencies, works with any unit testing framework.



##Node.js
A software platform that is used to build scalable network (especially server-side) applications. Node.js utilizes JavaScript as its scripting language, and achieves high throughput via non-blocking I/O and a single-threaded event loop.
Node.js contains a built-in HTTP server library, making it possible to run a web server without the use of external software, such as Apache or Lighttpd, and allowing more control of how the web server works.

##Express.js
Express is a minimal and flexible node.js web application framework, providing a robust set of features for building single and multi-page, and hybrid web applications. (Think Sinatra)

##Sails.js
Makes it easy to build custom, enterprise-grade Node.js apps. It is designed to mimic the MVC pattern of frameworks like Ruby on Rails, but with support for the requirements of modern apps: data-driven APIs with scalable, service-oriented architecture. It's especially good for building chat, realtime dashboards, or multiplayer games.



##jQuery
jQuery is a fast, small, and feature-rich JavaScript library. It makes things like HTML document traversal and manipulation, event handling, animation, and Ajax much simpler with an easy-to-use API that works across a multitude of browsers. With a combination of versatility and extensibility, jQuery has changed the way that millions of people write JavaScript.

##jQuery UI
jQuery UI is a curated set of user interface interactions, effects, widgets, and themes built on top of the jQuery JavaScript Library. Whether you're building highly interactive web applications or you just need to add a date picker to a form control, jQuery UI is the perfect choice.



##Underscore
A utility-belt library for JavaScript that provides a lot of the functional programming support that you would expect in Prototype.js (or Ruby), but without extending any of the built-in JavaScript objects. It's the tie to go along with jQuery's tux, and Backbone.js's suspenders.

##Two.js
A two-dimensional drawing api geared towards modern web browsers. It is renderer agnostic enabling the same api to draw in multiple contexts: svg, canvas, and webgl.



##Prototype.js
A JavaScript framework created by Sam Stephenson in February 2005 as part of the foundation for Ajax support in Ruby on Rails. It is implemented as a single file of JavaScript code, usually named prototype.js. Prototype is distributed standalone, but also as part of larger projects, such as Ruby on Rails, script.aculo.us and Rico. Prototype is used by 2.9% of all websites, which makes it one of the most popular JavaScript libraries. Prototype provides various functions for developing JavaScript applications. The features range from programming shortcuts to major functions for dealing with XMLHttpRequest.Prototype also provides library functions to support classes and class-based objects, something the JavaScript language lacks. In JavaScript, object creation is prototype-based instead: an object creating function can have a prototype property, and any object assigned to that property will be used as a prototype for the objects created with that function. The Prototype framework is not to be confused with this language feature.

##Rhino
An open-source implementation of JavaScript written entirely in Java. It is typically embedded into Java applications to provide scripting to end users. It is embedded in J2SE 6 as the default Java scripting engine.

##PhantomJS
A headless WebKit scriptable with a JavaScript API. It has fast and native support for various web standards: DOM handling, CSS selector, JSON, Canvas, and SVG.

##Moustache
Logic-less templates. AKA {{content}}. Found in many languages. Not actually Javascript in itself.

##Handlebars.js
Handlebars is a semantic web template system. It is a superset of Mustache, and can render Mustache templates in addition to Handlebars templates. Unlike Mustache, Handlebars add some logic to its templates. So it's not a logic-less template system. Used in Ember.js.

