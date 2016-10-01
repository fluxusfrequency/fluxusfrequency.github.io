---
layout: post
title: "How To Write Code With Style: 7 Tips For Cleaner Code"
date: 2016-08-29 20:29:38 -0600
comments: true
categories:
---

This post originally appeared on [Engine Yard](https://blog.engineyard.com/2016/how-to-write-code-with-style-7-tips-cleaner-code)

## Say What You Mean, Mean What You Say

Code is communication. It has two audiences: the computer, and the future maintainer. How we communicate with the computer is rather objective: you either gave it the instructions to do what you really wanted, or you didn't.

But people aren't so easy. As computer programmers, we channel our hopes, dreams, and moods into our writing. [WordPress](https://wordpress.org) popularized the slogan "Code is Poetry". To me, it reads as a true statement.

There are as many ways to write code as there are programmers, and we all do it a little bit differently. There's beauty in this diversity, but it also makes it harder to be understood. And the harder it is to read code, the longer it takes to maintain and extend it. Time is money, so this is a bummer when it comes to building a tech business.

How can we effectively communicate with other humans when we're programming? Write code with style! In this post, we'll explore seven rules of thumb that you can use to write cleaner code.

## 1. Business Value Comes First

There are many motivations that drive a programmer when she sits down to write code. Maybe she wants to be a [rock star](https://news.ycombinator.com/item?id=1757059). She's going to show everyone how smart she is by using the most arcane methods available in her language. Maybe she's interested in functional programming, and wants to see if she can find a legitimate use for [Proc#curry](http://ruby-doc.org/core-2.2.3/Proc.html#method-i-curry) in a production Ruby app. Or maybe she's ready for the weekend, and all that's on her mind is "ship it".

Whatever might be simmering below the surface, when we sit down to write code, we should take a step back and look at _why_ we're coding in the first place. Nine times out of ten, it's so that we can help make money for the band of misfits we run with. We should set aside our plans for world domination and adopt a view that asks "how can I help us succeed as a business?" After all, that's why we're writing code.

Code has to work. Preferably under all kinds of conditions. The better our codebase can withstand edge cases, the less likely we are to lose business. So we should strive to write the most solid instructions to the computer that we can.

## 2. Do What You Came To Do

If you've taken the time to open your editor and start writing code, there is a good reason that you're doing it. Whatever the specific problem you're trying to solve, it's best to stay focused on it. If your codebase is like most of the ones I've seen, there's probably a lot of touching going on between modules, classes, services, etc.

Sometimes when you're writing a feature, you can't resist going down a rabbit hole. You might think: "if only we used Active Model Serializers, I would be able to change this API response with a single line of code. I'll just do a quick refactor and put it in."

STOP. Maybe you're right. But do the thing you came to do. Make a note of your brilliant idea, and write a story for it when you're done with your feature so that you can give it your complete focus later.

Don't leave TODOs littered throughout the code with things you'd like to see done later (for example, `# Need to pull this out into a background job`). Write stories instead. That's what tracking systems are for.

Being present as you code can be threatened by many things. I've already mentioned tempting code changes. Other threatening distractions include social media, checking the news, Slack or HipChat, and growl notifications.

If you've never tried timeboxing (for example, with Pomodoros), I highly recommend it. Combine them with an app like [Freedom](https://freedom.to/) or [SelfControl](https://selfcontrolapp.com/) to keep yourself from getting pulled away, and pretty soon you'll find yourself able to avoid context switching for good blocks of time.

## 3. Keep It Simple, Stupid

You've heard the gospel before: premature optimization is evil. Don't begin by writing an abstract class and subclassing when you only have one use case! If I'm reading your code, I would much rather see magic strings and duplication between files than five levels of indirection between service objects that will take an hour to decipher.

When you're getting ready to introduce a new piece of functionality, try to solve it the easiest way you can think of first. It's classic TDD: red, green, refactor. That means when you're starting to solve a problem and you think "this is a _horrible_ way to do it", _stop yourself from trying to solve it the "right way"_. Write it horribly! Then you'll be able to step back and see exactly _why_ it's horrible, and with the help of the tests you wrote along the way, you can rewrite it in a cleaner way.

You also don't always have to worry about performance right out of the gate. Solve the problem first. If performance is _truly_ a concern, run some benchmarks, choose a reasonable solution, and document it so that the next person will understand what you're doing.

## 4. Backspace Is Your Friend

The number one thing that clients pay me for is understanding their business. I spend countless hours grokking custom DSLs, following paths of indirection, and deciphering other complexities of code. The mental overhead is usually high for getting things done. To some degree, this is unavoidable. But there is an inexpensive way to make code easier to understand, and therefore save money: delete unused code.

Don't be afraid to remove things that aren't getting used. If you're temporarily removing something, don't comment it out with a note like `TODO: put this back in after FooBar integration is complete`. If you're writing a bit of code that duplicates the functionality of something that's already there, pull out the old version when the new one is done. Just delete it. You can get it back, I promise. That's what Git is for.

Why should you delete it? When there's old, dusty code hanging around that never gets called, it confuses and scares developers. We don't know whether we are supposed to be using it, and we are afraid to delete it because we don't want to break anything.

If you have a decent test suite, you should be confident about pulling things out as they become unnecessary. If not, take the time to write a few integration cases. Just cover the workflows that are vital to your business. Doing so will pay huge dividends in the future, because you'll be able to remove dead code without fear, and keeping the project slim will make development faster in the future.

## 5. Be Understandable

Consider the following code:

```
(function(w, d, s, l, i) {
   w[l] = w[l] || [];
   w[l].push({
     'service.start': new Date().getTime(),
     event: 'service'
   });
   var f = d.getElementsByTagName(s)[0],
   j = d.createElement(s),
   dl = l != 'dataMonitor' ? '&l=' + l : '';
   j.async = true;
   j.src = '//cdn.foobar.com/script.js?id=' + i + dl;
   f.parentNode.insertBefore(j, f);
 })(window, document, 'script', dataMonitor, id);
```

Wat?

Just because you can write things succinctly doesn't mean you should. Code golf is fun as an exercise, but it's not fun to try to understand your minimal code when I'm trying to build a business. You should name things with full words that accurately describe what is happening.

Writing code that's easy to understand is both courteous and economical. I can't count the number of hours I've spent trying to decipher terse and cryptic blocks of code. It may seen on the surface like it doesn't matter how you write code _as long as it works_. But when you consider the number of hours saved across a team when code is easily understood, it's clear that how you write it can actually make a big difference.

## 6. Follow The Style Guide

Conventions are ways of writing that lessen the burden on the reader of having to figure out what the author intended to say. All writers have a relationship to conventions, whether it's strict adherence, defiance, or complete ignorance.

To communicate with an audience (which you are doing when you write code), knowing the conventions of your language is of the utmost importance. Writers of prose follow the strictures of Strunk and White's [Elements of Style](http://www.amazon.com/The-Elements-Style-William-Strunk/dp/1557427283).

By the same token, programmers should have a basic understanding of the norms in their community. Rubyists have the [Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide). JavaScripters are more contentious about conventions, but  the [AirBnB styleguide](https://github.com/airbnb/javascript) and Doug Crockford's [JavaScript: The Good Parts](http://shop.oreilly.com/product/9780596517748.do) are popular references. Those writing Python can run [Pep8](https://www.python.org/dev/peps/pep-0008/) to make sure their formatting is on point automatically.

Whatever your world, pay attention to the community around you. If you conform to what people expect, you're likely to get a lot more accomplished.

## 7. Always Include Documentation

As a consultant, I spend a lot of time reading through other people's code. Every few months I find myself in a new context, and unless I'm building a greenfield project, I have to ramp up on a codebase with months to years of history.

It's always surprising to me the extent to which teams rely on undocumented knowledge. When I'm trying to get a project up and running, I always have to reach out to somebody else at the company to get past a certain snag in the set up process. The inevitable answer comes: "oh yeah, you just need to _foobar_ the _bazqux_, then it will work."

While this process does get me where I need to go eventually, a lot of time and context-switching could be saved if the necessary hack were included in the README. With clear documentation, I probably could have figured it out on my own.

The same thing goes for comments. In communicating the purpose complex function, class, or method, your first line of defense should be good naming and clearly written code. But when the going gets rough, a few well-placed comments can go a long way.

Whatever you do, leave a trail of docs behind you. Doing so is a boon to the future maintainer of your code. And who knows, it might be you!

## Write Code With Style and Compassion

It's hard to understand code. The developer that comes after you will have to dig through a complicated folder structure to find your files, and when they get there they'll have to decipher your variable names, dig through your dependencies to find magic values, _and_ try to grok how what you've written stands for a real business situation.

To those of us writing modern, flexible languages, the possible ways to solve a problem are quite numerous. But we should realize that if we want to do our best to make our thoughts clear to whoever comes after us, our best bet is to write code as straightforwardly as we can.

We should strive to be humble. Leave behind the desire to write the most clever or efficient algorithm if it means sacrificing clarity. Blazing fast code has a time and place. But generally speaking, the money your company will save when the next programmer understands your code immediately will more than make up for the hundred milliseconds you would have saved by using bitmasking instead of a dictionary.

## Conclusion

In this post, we've talked about seven things to consider integrating into the approach you take to writing code. It's my hope that you weigh them, and perhaps call them to mind as you're programming. If we all do our part by choosing to write code with style, the codebases of the world will be a little cleaner, and working with computers will be a little more fun for us all.

Until next time, happy coding!

P. S. If the topic of code and communication interests you, I highly recommend checking out Matt Ward's excellent post [The Poetics of Coding](https://www.smashingmagazine.com/2010/05/the-poetics-of-coding/).

