---
layout: post
title: "Why We Use Test-Driven Development (TDD)"
date: 2016-02-29 05:25:52 -0600
comments: true
categories:
---

When you’re getting ready to build an application, there are many choices to be made. Will you choose to build it as cheaply as possible, then hope that things work out down the line? Or do you need something more resilient?  If so, spending the time and money to build a quality application will pay off.  Code written with craftsmanship will withstand the tests of time, changing business requirements, and user error.

At Quick Left, we believe that it's worth taking the time to build things right. A well-written codebase saves time and money, and withstands the test of time. On the other hand, we're also interested in doing things efficiently. If there's a way to write quality code _and_ save time and money, that's the kind of service we want to provide for our clients. That's why we use Test-Driven Development.

## What Is TDD?

Test-Driven Development is an approach to writing software in which the developer uses specifications to shape the way she implements a feature. For short, we describe it as the "red-green-refactor cycle". Before writing any code adding new functionality to an application, she first writes an automated test describing how the new code should behave, and watches it turn red (fail to pass). She then writes the code to the specification, and the test turns green (it passes). Finally, the developer takes a little time to make sure that the code just written is as clean as possible (refactor).

If you'd like to get a sense of what Test-Driven Development looks like in action, read my post about [creating a custom Ruby gem](https://quickleft.com/blog/wrapping-your-api-in-a-custom-ruby-gem/), where I walk through the process step by step.

TDD has long been a favorite approach of organizations that follow [Extreme Programming](http://www.extremeprogramming.org/) and Agile principles. In 2003, Kent Beck wrote [the book](http://www.amazon.com/Test-Driven-Development-By-Example/dp/0321146530) on this approach. In the years since, TDD has enjoyed enormous success, being elevated to near-dogmatic status.

In the past couple years, some have raised concerns that the programming community is being too strict in its demand for TDD. Most notably David Heinemeier Hansson (creator of Ruby on Rails), wrote a blog post entitled [TDD is dead. Long live testing.](http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html), which was met with passionate reactions from people on both sides of the argument.

At Quick Left, we continue to use TDD in our daily work. Experience has shown that it's the fastest, most reliable way to set our clients up for success. TDD consistently produces well-structured codebases that are well-structured, resilient, and easy to make changes to as business requirements evolve.

## What Are The Benefits?

What are the specific ways that a test-driven approach can benefit your business?

1. It Works The Way You Wanted It To

If computer systems built themselves, they would probably be perfect. Unfortunately, computers need people to tell them what to do, which means humans have to communicate to get software written. If you've worked in tech for any length of time, you've probably experienced the frustration of a mismatch between what was envisioned in planning and how an app actually behaves in the wild.

TDD helps alleviate this problem, because the test serves as a specification for what the code to be written should _do_. As long as you're [writing good stories](https://blog.engineyard.com/2015/happy-sad-evil-weird-feature-planning), your development team should be able to build exactly what you asked for. If your team agrees to use [Acceptance Test-Driven Development](https://en.wikipedia.org/wiki/Test-driven_development#TDD_and_ATDD), you can even write tests that describe how you want it to work in plain English!

2. Your Code Will Read Like Poetry

Because [refactoring code](http://fluxusfrequency.github.io/blog/2014/01/10/refactoring-1-extract-method/) is a built-in step in TDD, you end up with a much cleaner codebase as you go. Apps built with TDD tend to have less duplication, fewer edge cases that aren't thought through, and a better overall architecture.

Why should you care if the code is pretty or not? It makes it easier to add features, fix bugs, and get up to speed. Every time you [hire a consultant](https://quickleft.com/) or new developer, they have to spend a certain amount of time understanding how the different parts of your application interact. This [ramping up process](https://quickleft.com/blog/ramping-up-developers-on-code/) takes much less time when the code is well-structured. Similarly, programmers can make changes (whether features or bugs) much more quickly when there's less mental overhead to struggle with.

3. You Can See Trouble A Mile Away

Whether you decide to adopt TDD for the quality behavior or the squeaky-clean code, you'll enjoy this nice side benefit as well: you'll have a comprehensive test suite! That means that just about every line of code in your application will be covered by a test, and the entire thing can be checked automatically in a matter of minutes.

The benefit of a comprehensive test suite is that it alerts you to changes early. For example, if your checkout flow stops charging users' credit cards (eek!), you'll know it right away, because the tests will fail. It also means that if somone makes a mistake and something doesn't work the way it was supposed to, it will be obvious. That's good, because it will give you a chance to fix it before it goes to production. If it becomes necessary down the road, you can even start a campaign of deep refactoring without fear, because you'll have an ironclad test suite that will remain green.

4. It Saves You Money

In my experience working with [our clients](https://quickleft.com/casestudies/), companies often spend a lot more money adding features to their application than is really necessary.

One of the main causes of slow development is miscommuncation between product owners and developers. Too often, we experience the reality of mismatched expectations, and it feels like this cartoon:

![](http://www.cvr-it.com/images/PM_Build_Swing.gif)

The PO asks for one thing, the developer builds something else. Then cycles are wasted going back to fix the feature, when it could have just been built correctly in the first place. By translating business requirements into specs before writing any code, you can ensure that what you want is actually what gets built.

Another huge time sink in development is an over-complicated codebase making it difficult to make changes safely. When code is complicated, it gets much harder to get anything done, because one little change over here can result in a big problem over there. When following TDD, developers can make changes with confidence and your QA team will catch fewer regressions.

Both of these time sucks cost our industry innumerable amounts of money every year. If we just adopted a TDD approach, much of this money could be spent on new innovations instead.

## Conclusion

Hopefully you now have a better idea of what the red-green-refactor cycle is, and why we use Test-Driven Development at Quick Left. There's no better way to save time and money _while_ making sure that you have a codebase that's maintainable, extensible, and resistant to change. Whatever you're cooking up, I hope you seriously consider making TDD a part of your company's culture, because the benefits are so numerous and strong!

Best of luck.
