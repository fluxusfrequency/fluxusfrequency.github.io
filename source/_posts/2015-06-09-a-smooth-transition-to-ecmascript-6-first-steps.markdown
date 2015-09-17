---
layout: post
title: "A Smooth Transition to ECMAScript 6: First Steps"
date: 2015-06-09 06:28:25 -0600
comments: true
categories:
---

This post originally appeared on [Engine Yard](https://blog.engineyard.com/2015/smooth-transition-ecmascript-6-integration).

## Introduction

I'm really excited about the newest version of JavaScript, *ECMAScript 6* (ES6). But I'm also terrified. There's already so much to do between mentoring, contributing to open source, and working on the projects that pay the bills. When will I ever find the time to learn a whole new version of JavaScript?

As developers, it's our blessing and curse to always be learning. When I think about getting ready to adopt ES6, I feel some anxiety about the thought of having to figure out all of the new patterns and APIs it exposes.

In this miniseries, we'll look at some quick and easy ways to integrate ES6 into what you're working on today. Hopefully, by adding ES6 patterns into our coding practice a little at a time, we'll be able to avoid spending a weekend learning the new API when we could be playing outside.

## Looking Ahead to ES6

ES6 will be the first update to JavaScript since ES5 was finalized in 2009. ES6 was originally slated to come out in 2013, but was then pushed back a couple more times, and is expected to be finalized this month, June 2015.

There's a lot of JavaScript in programming these days.

It's found on the server as [Node](https://nodejs.org/), in [OS X Automation](https://developer.apple.com/library/mac/releasenotes/InterapplicationCommunication/RN-JavaScriptForAutomation/), and of course, in all of the web browsers—where many of us spend most of our time writing JS apps. And we'll be writing ES6 in all of these locations before you know it. It's already standard in [Ember CLI](http://www.ember-cli.com/), [Angular 2.0 will be based on it](https://www.airpair.com/angularjs/posts/preparing-for-the-future-of-angularjs), and is [making its way into Node](https://github.com/joyent/node/wiki/ES6-%28a.k.a.-Harmony%29-Features-Implemented-in-V8-and-Available-in-Node) bit by bit.

Thinking ahead, it's clear that JavaScript developers will need to start learning ES6 sooner or later. The good news is, it's a superset of ES5, which means all the ways we currently write code will still work. So we can just write code the way we always have done, until we see places where we can use something from ES6.

## Compatibility

Before we get into the nitty-gritty of how to start using ES6, a quick note about compatibility. As of this writing, most JavaScript engines are in the process of implementing the features called for by the ES6 spec. To see a list of the features that are slated for release in ES6 (and how they compare with ES5), check out [this reference](http://es6-features.org/). If you want to read about specifically what's available right now, check out this [compatibility table](http://kangax.github.io/compat-table/es6/). In *this* miniseries, we're going to be focusing on the features that are currently available in the major ES6 engines.

There are many features that haven't been rolled out yet, but can be easily _transpiled_ (transformed and compiled) to ES5 for immediate use. There are several compilers and polyfills available to help with transpiling. My favorite is [Babel](https://babeljs.io/), formerly called 6to5. Babel 5.0 was released on March 31, 2015, and is currently leading other options, with 76% of the spec in place.

Regardless of which transpiler you use, there are several features that are still mostly unsupported across all JS engines. These include: tail calls, WeakMap, WeakSet, Proxy, Reflect, Symbol, new.target, and subclassing built-ins, among others.

Some of these features are available in Babel, but only with experimental mode turned on. For the purposes of this miniseries, we'll be looking at some of the most widely supported, easy-to-use features—so we won't be covering experimental features like these.

We're also going to skip over some of the features that are a little more difficult to get started with, such as iterators, generators, and proxies.

## Getting ES6 Into Your Build Process

Since we can transpile ES6 code to ES5 and start using it everywhere now, it doesn't hurt to get it set up in our build process, so that we can just start using it without having to think twice about it. Luckily, this is really easy to do with Babel.

It's beyond the scope of this miniseries to get into all the variations of what you might encounter in getting ES6 into your build process, but you probably won't have much trouble adding it as a step in your existing build process.

There's a Babel plugin available for just about every build setup, including Grunt, Gulp, and Broccoli, and whether you're using Browserify or RequireJS. It even has a built in [JSX transpiler](https://babeljs.io/docs/usage/jsx/), making it really easy to use with React. There's a full list of build tools and how to use them [on the Babel website] (http://babeljs.io/docs/using-babel).

You may also want to make use of the [babel-runtime](https://babeljs.io/docs/usage/runtime/) package. This is an optional transformer that prevents duplication of common functions during compilation. It also sandboxes your code, aliasing many globals to `core-js` to avoid polluting the global namespace.

### With Browswerify + NPM Scripts

Here's a look at how you might add Babel into an existing client-side JS project, using [Browserify](http://browserify.org/) and the [Babelify](https://github.com/babel/babelify) transform module.

In `package.json`:

```javascript
{
  "scripts": {
    "postinstall": "browserify --debug --standalone MyApp assets/js/index.js --transform [ babelify --optional babel-runtime ] --outfile build/my-app.js"
  },
  "dependencies": {
    "babel-runtime": "^5.0.12",
    "babelify": "^6.0.2",
    "browserify": "^9.0.7"
  }
}
```

Note that by defining the build script under the `postinstall` property, it will be run automatically after the package is installed. This might be handy when deploying to [Engine Yard](https://www.engineyard.com/), as it will prevent you from having to explicitly call a build in your deploy script.

## Let's Do It

Now that you've gotten ES6 set up in your build, you can just start writing it whenever you're developing. Or, if you don't feel like it, you can just fall back to ES5. Remember, it's all valid ES6!

In this post, we talked about the upcoming transition to ES6 everywhere that JavaScript is written. We’ve looked at which ES6 features are currently supported and how to get ES6 into your projects so you can start using it right away.

Make sure to tune tomorrow for part two of this miniseries, where we’ll walk through some of the easiest places to start using ES6 in a typical front-end [Backbone + React](https://blog.engineyard.com/2015/integrating-react-with-backbone) project.

Until then, happy hacking!
