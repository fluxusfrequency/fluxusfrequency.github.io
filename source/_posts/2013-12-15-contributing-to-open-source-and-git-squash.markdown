---
layout: post
title: "Contributing to Open Source &amp; Git Squash"
date: 2013-12-15 21:01
---

## Open Source

I've been really interested in contributing to open source lately. In part, because it seems like a really good way to learn about the gems I use. Also, because I am on a campaign of giving right now in my life, and contributing is an easy way to do so in computer code (which is basically all I do/breathe/sleep right now).

I started off with something small, a contribution to the [Ember.js guides](http://emberjs.com/guides/). I noticed that there was an inconsistency in one step of the tutorial: sometimes they called the everyProperty() method, and other times they called everyBy(). I corrected it to use everyProperty() throughout and submitted. Even though my pull request was not accepted, because the latter method was deprecated in favor of the former, it was a magical experience. The people working on Ember were talking to me! And I was helping them make Ember better! I was feeling into it.

Over the next month or so, I submitted some other documentation pull requests. I had pull requests accepted to Sinatra, Express.js, and even Rails! After being scolded for not including "\[ci skip]" in my commit message (to prevent the continuous integration build from running for a simple documentation fix), I got a tip in the form of less than 1% of a Bitcoin. This was getting exciting.

Emboldened, I decided to try for something bigger. As I was reading Addy Osmani's [Developing Backbone.js Applications](http://addyosmani.github.io/backbone-fundamentals/), I noticed that a portion of the chapter on Marionette was not working. I dug into the source code for the book and for the Marionette example, and discovered they didn't match. I updated the book and submitted a pull request. Several days later, it was merged. I contributed to a book! Cool!

About this time, I saw that one of my classmates' mentors, Austen Ito, was doing the [24 Pull Requests](http://24pullrequests.com/) challenge, submitting a pull request every day in December up until Christmas. I decided to jump on board.

In my pursuit of Rails Ninja status, I'd been working on learning how to use engines. Ryan Bigg's [Multitenancy with Rails](https://leanpub.com/multi-tenancy-rails) and José Valim's [Crafting Rails 4 Applications](http://pragprog.com/book/jvrails2/crafting-rails-4-applications) got me off to a good start. An excellent start, actually. I decided to go further with my favorite source for Rails research, [The Rails Guides](http://guides.rubyonrails.org). I was disappointed to find that the engine guide was pretty hard to read, mostly due to a lot of run-on sentences. I decided to edit the whole thing as I went.

## Git Squash

Things weren't as easy with this pull request. As I went through and made spelling and grammar fixes, I also removed some word wrapping. I submitted the pull request. The Rails contributors on Github were very nice, but were hoping I could change about seven things. I did, and submitted a new commit. Then they asked me to fix the wrapping in a separate commit, so I did that. Now I was up to four commits, when I should really only have two.

How did that git squashing thing work again?

I could have created a new branch, cherry-picked my commits, and submitted a new pull request. But I was feeling stubborn, and wanted to figure out squashing. I asked many of my friends and teachers. We tried `git rebase -i`, like all of the articles we found online suggested, but it wasn't working. I'd either end up with the same commits, or lost in some kind of detached head rebasing state that I couldn't resolve. [My favorite guide to git](http://www.ndpsoftware.com/git-cheatsheet.html) was no help. Even `git reflog` wasn't pointing the way out.

In the end, Katrina Owen came to the rescue (via email, from home). Here was my message to her:

```plain

Hey Katrina,

Thanks for helping. I was trying to use git rebase -i for this, but not getting the results I wanted.

I'm contributing edits to the rails guide for engines. I was asked to revise my original commit, then squash it. I was also asked to send a separate commit with word wrapping.

Here's the history of the branch I'm working on:

* 0cdcf46 2013-12-11 | Word wrap to engines guide \[ci skip] (HEAD, patch-4) [Ben Lewis]
* 492ac2a 2013-12-11 | Revision to guide edits (origin/patch-4) [Ben Lewis]
* b55a803 2013-12-04 | Clarification, grammar fixes, punctuation, and capitalization \[ci skip] [Ben Lewis]
* 49ed8a1 2013-12-04 | Run Travis tests using Ruby 2.1.0-preview2 too [Santiago Pastorino]

I need to squash them so that I only have 0cdcf46 and 492ac2a (although I would like to take the message from b55a803). Here's what I want to see:

* 0cdcf46 2013-12-11 | Word wrap to engines guide \[ci skip] (HEAD, patch-4) [Ben Lewis]
* 492ac2a 2013-12-11 | Clarification, grammar fixes, punctuation, and capitalization \[ci skip] [Ben Lewis]
* 49ed8a1 2013-12-04 | Run Travis tests using Ruby 2.1.0-preview2 too [Santiago Pastorino]


What should I do?

Thanks!

```

Here's what she replied:

```plain

A separate commit, or a separate pull request?

If it's a separate pull request, here's how I'd do it:

Create a separate branch from the most recent upstream master,
and then do:

   git cherry-pick 0cdcf46

Then you can submit that branch as a separate pull request.

On your branch that you've been working on, you could get rid
of the most recent commit with

   git reset --hard HEAD~1

Then you can rebase -i with:

   git rebase -i HEAD~4 # or something

Next to the second commit that you want to squash into the first commit,
replace 'pick' with 'squash'.

Then write and quit, and it will re-open a new editor and ask you to fix
the message. It will give you both of the previous ones, and you can
pick one or the other or write a whole new message.

```

It was an easy fix: just use the HEAD~X for the number of commits you want to work with. Worked like a charm! I was able to squash the commits just as I wanted, and resubmit them. Next up, I had to remove the wrapping changes from the first commit (b55a803). This was a good chance to practice squashing again. This time around, I did end up using some cherry-picking magic from a separate branch, but I was still able to squash my commits. Success!

It's crazy how hard it has been to get used to git. Probably, a lot of the overwhelm comes from [all of the other things](https://www.codefellows.org/blogs/this-is-why-learning-rails-is-hard) I have been learning at the same time. Anyway, now I know how to squash.

## Moving Forward

I'm really interested in contributing some actual code to projects. I've been looking around at the issues on Rails, Sorcery, Mechanize, and other popular gems, but I often find that they're way over my head. I was able to fix a small todo in the [Exercism](http://www.exercism.io) source code, but beyond that, I'm pretty much lost. If anyone reading this has any good leads for something that an advanced beginner can do (somewhere between documentation edits and http response parsing level insanity) please hit me up on Twitter!

