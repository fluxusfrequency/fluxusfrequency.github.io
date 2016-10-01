---
title: 'Bring Back Your App: Ramping Up Developers On Code'
date: 2015-10-06 02:52:27 Z
categories:
- source
layout: post
comments: true
---

This post originally appeared on [The Quick Left Blog](https://quickleft.com/blog/ramping-up-developers-on-code/)

## Introduction

Remember the glory days? Your company had it all! New signups every day. Coverage on the hottest blogs. Money was rolling in hand over fist. All thanks to your hot app. You and your goons really nailed it when you followed an [MVP](https://quickleft.com/blog/actually-mvp/) process and built just the thing the market was looking for.

But then something happened. You shifted focus to integrating with another company's API. Or you lost your entire developer team. Slowly, kudzu vines covered over what was once a glorious app. The technology you used to build grew outdated. Bugs accumulated, but there was no time to fix them. You shed a single tear when you thought back to the glory days.

Finally, a new day dawned! You got a funding round! You've just hired a new team of developers. They start next week. You can't wait to get them slinging code. But are you ready? Starting a new team on an old app isn't as easy as handing them laptops and saying "clone down the repo". In this post, the first of a two-part series, we'll take a look at some advice you can use when ramping up developers on code in a legacy application. When we're through, you'll be ready to let the good times roll again!

## Have An Easy Setup Process

Getting developers set up on a project can be time-consuming. Maybe you're planning to write off two days for each person to set up their laptop, get the application and its dependencies installed, and familiarize themselves with the codebase. If it's just one or two developers, you can probably justify four days of lost time. But what if you're bringing on a team of ten? And what if they could get set up in one day? Or even half a day? That would be 15 days spent on developing new features and bringing in more money!

So how can you get them up and running faster?

### Documentation

Documentation is _so_ important. I can't possibly recommend it enough. As a consultant, it's one of the first things I look for when checking out a new code base. Docs are the most efficient way to get a new team member up and running. If they're up-to-date, new devs can _read_ about the project architecture and setup process, instead of having one of your more experienced developers take the time to sit down and explain it.

There are many effective ways to make documentation accessible to your team. I've seen companies have success with their [GitHub project wiki](https://help.github.com/articles/about-github-wikis/), [Atlassian Confluence](https://www.atlassian.com/software/confluence), a `docs` folder in their app repo, or a separate `docs` repo on GitHub.

Document as much as you can. Your main application should have a thorough `README`. It's the first thing people will see when visit the GitHub repository. If you're running separate server and front-end apps, there should be a way to understand what has to be done to get them to talk to each other in development. How do you install the dependencies? How do you run the tests?

Sometimes, project setup can get held up by weird system setup issues. Maybe the app used to run on a pre-Yosemite version of OS X, but when you try to install it on Yosemite, it runs into problems with Nokogiri. I recommend creating a `troubleshooting` document and putting errors and their fixes into it. At [Quick Left](https://quickleft.com/), we cut our dev setup time by several hours when we introduced a `troubleshooting` doc in the [Sprint.ly](https://sprint.ly/) repo. It also allowed senior members of the team to stay focused, instead of context switching to help people.

Some other good things to document include: language style guides, office culture, [PR templates](https://quickleft.com/blog/pull-request-templates-make-code-review-easier/), things to look for in PR review, QA tasks, and deploy steps.

### Docker

We live in an exciting time. The dawn of containers has made it easier than ever to make sure that each person working on a project is working with the same system setup. I like to use [Docker](https://www.docker.com/) to facilitate running applications across all environments, from development to CI to production. If you've never created a container before, I recommend checking out my colleague Alex Johnson's [Sailing Past Dependency Hell With Docker](https://quickleft.com/blog/sailing-past-dependency-hell-with-docker-distributed-systems/).

Docker isn't right for every team, but if you have the knowledge and time to get it set up, it can save hours of time that would otherwise be spent googling errors and entering arcane commands into SSH tunnels. I'd recommend looking into it.

## Pay Off Technical Debt

Great documentation is a good first step, but often it isn't enough to ramp up developers on code in a reasonable amount of time on its own. If your app has accrued significant amounts of technical debt, this can present a major milestone. There are two kinds of tech debt to look for: dependency debt and native debt. Here are some ways to pay down each.

### Dependency Debt

If your app was built back in the heyday of _(name your tech stack here)_, you probably made use of a lot of open source packages that were popular and well-maintained at the time. But fast-forward a few years, and suddenly some of your dependencies are outdated, others deprecated. This can be not only frustrating, but also a potential security concern.

If this describes you, you've probably missed some patches that deal with widely publicized security issues. This can be a concern for open source packages and languages alike. For example, if you're still running Ruby 2.1.1, you might find that you're vulnerable to DNS Hijack attacks and [other security holes](http://www.cvedetails.com/vulnerability-list/vendor_id-7252/product_id-12215/version_id-164073/Ruby-lang-Ruby-2.1.1.html).

It's safest to invest a couple of days getting your language and dependency versions updated _before_ bringing on a bunch of new developers. Not only will this fix your security holes, it will save you tons of time in working out module version conflicts.

#### Gems & Modules

If your app is built in Ruby or JavaScript, you're probably making use of [Bundler](http://bundler.io/) or [NPM](https://www.npmjs.com/) to resolve dependency versions and pull them down. In your `Gemfile` or `package.json`, you may be specifying the specific versions of packages you want to use, like this:

```ruby
gem 'rails',                  '~> 3.2.17'
gem 'jquery-rails',           '~> 2.1.3'
gem 'mysql2',                 '~> 0.3.11'
gem 'devise',                 '~> 2.2.0'
gem 'cancan',                 '~> 1.6.8'
gem 'nokogiri',               '~> 1.5.6'
```

Or this:

```javascript
"dependencies": {
  "config": "^1.10.0",
  "hapi": "8.1.0",
  "hapi-auth-cookie": "^2.0.0",
  "superagent": "^1.2.0"
}
```

Your new developers pull down the repo, run `bundle install` or `npm install`, and soon find out that you actually _can't_ install some things on the latest version of OS X without passing special flags.

Or maybe you realize that the authentication gem you're using is one or two major versions behind the latest release. Fine, you think, and update the version specification. But then, when you try to run the install, you have version conflicts with another gem.

I have yet to find a good way to resolve all of these problems at once. The best approach I've found is to start with the module that you most need to update. Bump its version, then try running the `install`. If there's a dependency version conflict, change that. Keep following this pattern until the waterfall of pain subsides. Then repeat the process with the next package that you know needs updating. If you run into issues, sometimes a `bundle update` or deleting the `node_modules` folder completely and reinstalling can get you closer to success.

#### The Rails Upgrade By Version Trick

When it comes to updating Ruby gems, I've come across nothing more painful than updating [Rails](http://rubyonrails.org/) across multiple versions. I've recently been working on upgrading an old Rails 3.1 app to Rails 4.2 for a client. When I try updating the gem version all at once, I get so many regressions and new bugs that I don't even know where to start.

There is a better way. Begin by upgrading Rails one _minor_ version at a time. So, if you're on 3.1, upgrade to 3.2 before attempting to go to 4.0. Then go on to 4.1 and 4.2. There's a [list of Rails releases](https://rubygems.org/gems/rails/versions) you can use to drive this process. I've also found this [Rails Upgrade Checklist](http://www.rails-upgrade-checklist.com/) site pretty helpful. It lets you know all the deprecations you need to change for each version along the way.

### Native Debt

So you've got all of your dependencies updated, and installing your app is a breeze. You can get a new developer from zero to sixty in an hour. Great! But now, do they actually understand anything about what's going on in the code base?

If you're lucky, your last team wrote clear, understandable code with lots of comments and no duplication. In reality, the odds of that are pretty low. Because of customer demands and pressure on time and budget, every code base ends up with some cruft, duplication, and just plain cryptic parts.

Take the time before your new team rolls on to clean things up a little bit. It will be invaluable in getting them up and running, ready to be part of the conversation on architectural decisions. Here a couple of steps you can take to get there.

#### Refactor

You've heard it before: if you practice agile development, you need to refactor. It's something you should be doing along the way if you're practicing Test Driven Development. The mantra goes: `red, green, REFACTOR`. Nevertheless, there are probably dozens of places that your code could benefit from some refactoring.

If your team is new to refactoring, I highly recommend the [Refactoring book](http://martinfowler.com/books/refactoring.html) or its [Ruby counterpart](http://martinfowler.com/books/refactoringRubyEd.html). You don't have to become refactoring experts, just pick up a couple of patterns and go to town. Most of the time, there is some kind of smell that's repeated throughout the codebase. If you learn to identify it, it becomes easy to identify and snipe.

Generally speaking, I've found that the easiest wins are to `extract` and `inline` code. This works at any level of scope. If you have a big, unweildy method that's doing a lot, you can [extract a method](http://fluxusfrequency.github.io/blog/2014/01/10/refactoring-1-extract-method/) from it. If you have a class calling a method that doesn't make any sense, you might want to [extract an object](http://fluxusfrequency.github.io/blog/2014/01/17/refactoring-2-replace-method-with-method-object/). You can also go the other way: if there's some unnecessary indirection, you can pull code right into the class or method that needs it.

Spending a day or two refactoring the hairiest parts of you application can make things  understandability. That makes a big difference in your team's  efficiency.

#### Delete Dead Code

Related to refactoring, another great way to reduce the mental overhead involved in starting with a new app is to remove dead code.

One easy win is to "snipe" unused dependencies. You should also look through the codebase and remove any functions or methods that are old and not being used anymore. It can be hard to determine where exactly those spots are. If you have someone who's familiar with the code, asking them can be a good place to start.

My colleague [Meeka Gayhart](https://quickleft.com/blog/author/mgayhart/) also suggests cleaning up cruft by looking for all the places where a package or method is called, and removing it if there aren't any callers in the current code. You can also look through git history to confirm using the [git pickaxe](http://www.philandstuff.com/2014/02/09/git-pickaxe.html).

If you have a good test suite, make sure to run it often as you remove suspect bits of code to make sure nothing broke.

Take the time to clean up your code base before new developers roll on, it can save huge amounts of time. New devs don't have to work as hard to understand what's going on, so they can spend those mental cycles shipping new features instead.

## Wrapping Up

In this post, we looked at a couple of strategies for getting a new team ready to develop on an old code base. We looked at how to streamline the technical set up process, some tips for updating dependencies, and ways to pay down technical debt.

There are few things more exciting to a major stakeholder in an application than seeing it rise from the ashes to conquer the world a second time. I hope these tips help get you back to the glory days! Stay tuned for part two of this series, where we'll take a look at some higher-level planning ideas that can help ensure you make the most of your development time from here.

