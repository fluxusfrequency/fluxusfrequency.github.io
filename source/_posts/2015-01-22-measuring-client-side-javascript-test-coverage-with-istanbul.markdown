---
layout: post
title: "Measuring Client-Side JavaScript Test Coverage With Istanbul"
date: 2015-01-22 21:16:59 -0700
comments: true
categories:
---

This post originally appeared on [Engine Yard](https://blog.engineyard.com/2015/measuring-clientside-javascript-test-coverage-with-istanbul).

It was also published on the [Quick Left Blog](https://quickleft.com/blog/measuring-clientside-javascript-test-coverage-istanbul/).

## Introduction

Testing is a vital part of any development process. Whether a project's authors are hoping for scalability, fewer bugs, or just code that stands the test of time, a solid test suite is essential.

It's all well and good writing tests, but how do you know that you've tested the code that matters? Did you cover the most important components? Did you test [the sad path](http://engineering.imvu.com/2012/05/08/writing-resilient-unit-tests-3/)? The edge cases? What about the [zero states](http://emptystat.es/)?

Using [Istanbul](http://gotwarlost.github.io/istanbul/), you can generate a nice coverage report to answer these questions. In this article, we'll look at how to get it set up in a typical clientside JS project.

## A Note About Metrics

Before we get into the process of setting up Istanbul, I'd like to talk about code coverage as a metric. Metrics in programming can be very two-sided. On the one hand, they can be very useful in measuring velocity and predicting completion of new features. But, they can also become self-fulfilling prophecies leading to bad code.

For example, if a project manager or tech lead measures her programmers' effectiveness by counting the lines of code they write, she will not find that those who wrote the most lines are the ones that wrote the best code. In fact, there's a danger that some of the programmers will adopt a verbose style, using far more lines than necessary to get the same thing done, in order to bump their ranking.

Placing a strong emphasis on test coverage as a metric can lead to a similar problem. Imagine that a company adopts a policy that any pull request must increase or maintain the percentage of lines tested. What will be the result?

I imagine that in many cases, developers will write good tests to accompany their features and bugfixes. But what happens if there’s a rush and they just want to get the code in?

Test coverage tools only count the lines that were hit when the test suite was run, so the developer can just run the line in question during the test, while making a trivial assertion. The result? The line is marked covered, but nothing meaningful is being tested.

Rather than using test coverage as a measure of developer thoroughness, it makes a lot more sense to use coverage as a way of seeing which code _isn't_ covered (hint: it's often `else` branches). That information can then be used to prioritize testing goals.

In summary: don't use code coverage to measure what's tested, use it to find out what isn't.

## Time to Test

Let's imagine that we have a clientside JavaScript application written in Angular, Ember, or Backbone and templated with Handlebars. It's compiled with Browserify and built with NPM scripts.

This application has been around for a couple of years, and due to business pressures, its authors have only managed to write a handful of tests. At this point, the app is well-established, and there _is_ a testing setup in place, but there's also a lot of code that's untested.

Recently, the company behind the application closed a funding round, and they're feeling flush. We've been brought in to write some tests.

Because we're hotshots and we want to show off, we decide to begin by taking a snapshot of the current code coverage, so that we can brag about how many percentage points we added to the coverage when we're done.

## Setting up Istanbul

This is where [Istanbul](http://gotwarlost.github.io/istanbul/) comes in. Istanbul is a code coverage tool written in JavaScript by [Krishnan Anantheswaran](https://github.com/gotwarlost) of Yahoo!.

It can be a little tricky to set up, so let's take a look at one way to do it.

There are four steps in our approach:

1. Instrument the source code with Istanbul
2. Run `mocha-phantomjs`, passing in a `hooks` argument
3. Use a `phantomjs` hook file to write out the results when testing is complete
4. Run the `Istanbul` cli to generate the full report as an HTML file

Let's get started.

### 1. Instrument the Source

We'll need to find a way to run our code through Istanbul after it's been compiled, so the first step is to set up an NPM task that will pipe compiled code into a tool like [browserify-istanbul](https://github.com/devongovett/browserify-istanbul).

Just in case you're not using `browserify`, a variety of other Istanbul NPM packages exist for instrumenting code, including [browserify-gulp](https://github.com/SBoudrias/gulp-istanbul), [grunt-istanbul-reporter](https://github.com/justspamjustin/grunt-istanbul-reporter), and [borschik-tech-istanbul](https://github.com/bem/borschik-tech-istanbul).

For the moment, let's imagine that we _are_ using Browserify. We already have NPM tasks in place to compile our code for development/production and for testing. Here's what they look like. Note that the `-o` option to `browserify` specifies the output file for the build and the `-d` option turns on debugging.

In `package.json`:

```javascript
{
  ...
  "scripts": {
    ...
    "build": "browserify ./js/main.js -o html/static/build/js/myapp.js",
    "build-test": "browserify -r handlebars:hbsfy/runtime ./js/test/index.js -o html/static/build/js/test.js -d --verbose"
  }
  ...
}
```

We can use the `build-test` task as a template for a new `build-test-coverage` task.

Before we do that, we'll want to make sure we pull in `browserify-istanbul` with `npm install --save-dev browserify-istanbul`.

Next, we'll write the task in `package.json`. We'll ignore the Handlebars templates and Node modules when we load everything into Istanbul. We'll also use the `-t` option to Browserify to use a [transform module](https://github.com/substack/module-deps#transforms).

```javascript
{
  ...
  "scripts": {
    ...
    "build-test-coverage": "mkdir -p html/static/build/js/ && browserify -r handlebars:hbsfy/runtime -t [ browserify-istanbul --ignore **/*.hbs **/bower_components/** ] ./js/test/index.js -o html/static/build/js/test.js -d"
  }
  ...
}
```

With the `browserify-istanbul` package and the `build-test-coverage` script in place, we've got our code instrumented, and we're ready to move on to step two.

### 2. Run `mocha-phantomjs`, Passing in a Hooks Argument

Now that the code is all built and ready to go, we need to pass it into a test framework. We'll use write a cli script that spawns [mocha-phantomjs](https://github.com/metaskills/mocha-phantomjs) in a child process. We'll pass in a hooks argument that specifies a `phantom_hooks.js` file we've yet to write.

(Note: if you're using `gulp` or `grunt`, you may want to check out [gulp-mocha-phantomjs](https://github.com/mrhooray/gulp-mocha-phantomjs) or [grunt-mocha-phantomjs](https://github.com/jdcataldo/grunt-mocha-phantomjs) for this step.)

In `js/test/cli.js`:

```javascript
#!/usr/bin/env node

var spawn = require('child_process').spawn;

var child = spawn('mocha-phantomjs', [
  'http://localhost:9000/static/js/test/index.html',
  '--timeout', '25000',
  '--hooks', './js/test/phantom_hooks.js'
]);

child.on('close', function (code) {
  console.log('Mocha process exited with code ' + code);
  if (code > 0) {
    process.exit(1);
  }
});
```

With our cli script in place, we've now got a way to put our Istanbulified code into PhantomJS, so we'll move to step three.

### 3. Use a PhantomJS Hook File to Write the Results

In the script we wrote in the last section, we passed a hooks file to `mocha-phantomjs`, but we hadn't created it yet. Let's do that now.

After all the tests have run, our hook will grab the `__coverage__` property of the window, which contains the result of our coverage run, and write it to a `coverage/coverage.json` file.

We'll load this data into the Istanbul CLI to generate a more readable report in the next step.

In `js/test/phantom_hooks.js`:

```javascript
module.exports = {
  afterEnd: function(runner) {
    var fs = require('fs');
    var coverage = runner.page.evaluate(function() {
      return window.__coverage__;
    });

    if (coverage) {
      console.log('Writing coverage to coverage/coverage.json');
      fs.write('coverage/coverage.json', JSON.stringify(coverage), 'w');
    } else {
      console.log('No coverage data generated');
    }
  }
};
```

With our coverage data saved, we're ready to move on to the last step.

### 4. Run the Istanbul CLI to Generate the Full Report

The final step in our process is to take the coverage data generated in step three and plug it into the Istanbul CLI to generate a coverage report HTML file.

We'll write an NPM script that executes the `instanbul report` command, passing it the folder where we saved our results (`coverage`), and specifying `lcov` as our output format. This option will save both `lcov` and `html` files.

```javascript
{
  ...
  "scripts": {
    ...
    "coverage-report": "istanbul report --root coverage lcov"
  }
  ...
}
```

Now we have all the scripts we need to generate and view our coverage report.

## Generating the Report

We'll need to run each of the commands that we've defined before we'll be able to view our results (you may want to wrap these in a single NPM task for convenience).

- `npm run build-test-coverage` to compile our test code and load it into Istanbul
- `./js/test/cli.js` to run the tests with `mocha-phantomjs` and write the `coverage.json` file
- `npm run coverage-report` to format the coverage results as an HTML file

Once we've completed these steps, we should see a new `coverage/lcov-report` folder containing an `index.html` file and a few assets to make it look pretty.

## Viewing the Report

If we open the generated file in our browser, we'll see an easy-to-read breakdown of our coverage.

There are four category columns at the top of the page, each telling us about a different aspect of our coverage.

- *Statements* tells up how many of the statements in our code were touched during the test run.
- *Branches* tells us how many of our logical `if`/`else` branches were touched
- *Functions* tells us how many of our functions were touched
- *Lines* tells us the total lines of code that were touched

There's also a list of the folders in our project, each with a categorical coverage breakdown. You can also click on each of the folders to further break down the coverage file by file. If you then click on a file, you'll see its contents highlighted in green and red to indicate covered and uncovered lines.

The ability to zoom in and out on our project and see the categorical breakdown at each level makes Istanbul particularly nice to work with.  It also makes it easy to dig in and explore places in your code base that might benefit from some additional testing.

## Wrapping Up

If you haven't added code coverage to your JS projects yet, I highly recommend it. Getting everything up and running is a minimal time investment, and it can really pay off.

Although I would discourage you from measuring the success of your test suite based on percentage points only, getting an in-depth look at what you currently are and aren't touching can really pave the way to uncovering bugs you didn't even know you had.

I hope that this post has helped you get Istanbul up and running with a minimum of heartache. Until next time, keep testing!
