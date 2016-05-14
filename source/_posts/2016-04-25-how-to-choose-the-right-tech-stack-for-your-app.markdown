---
layout: post
title: "How To Choose The Right Tech Stack For Your App"
date: 2016-04-25 05:26:07 -0600
comments: true
categories:
---


Whether you're bootstrapping a startup or the overseer of a tech empire, there comes a time when you have to make tough decision. You're getting ready to build (or rebuild) a new application that will be vital to your company's continued success. What tools should you use?

As consultants at Quick Left, clients ask us to help them make this decision regularly. We've worked alongside many businesses as they found their way to success. We've also worked with a wide array of languages and frameworks. One thing we've learned from these experiences is that using the wrong technology can set you back by weeks or months at best, which is both costly and risky from a business point-of-view. On the other hand, placing a good bet makes it easier to scale and grow.

There are a lot of things to consider when trying to choose the right tech stack for your app. In this post I'll walk you some of the things I usually consider when making a recommendation.

## Some Basic Things to Consider

The first questions to ask yourself when making this decision are the same ones you would consider when thinking about your [User Experience](https://quickleft.com/blog/ux-is-not-just-ui/). Before you decide _how_ you'll build your app, you'll want to decide what _kind_ of app you're building.

Who is your user base? What's their demographic? Are they young? Old? Urban? Rural? Comfortable with technology? More importantly, do you expect them to access your application from their mobile device, desktop computer, or something else? Knowing whether you need a mobile or web application, or both, will narrow down your choices of tech quite a bit.

Another consideration is whether what you are making is very similar to an existing application, a riff on a known paradigm, or something the world has never seen before. If your customers are used to a native mobile experience, you will probably want to choose a native technology, such as Objective C, Swift, or Java. If you're trying to bring a desktop-style experience to the web, you'll probably be looking at a single page app with a JavaScript heavy front-end and a modern framework like [React](https://facebook.github.io/react/) or [Angular](https://angularjs.org/).

Finally, think about your timeline. Do you need to get to market as fast as possible, or do you have time to build something with a little more lasting power? Some technologies, like WordPress and Ruby on Rails, are great for getting a product out the door as quickly as possible. Others, like Go and Scala are great at solving certain problems like scalability and performance, but will take a little longer to build.

Answering these basic questions about your business will help narrow down the technologies you need to look at. After you've thought about them, I recommend considering the character of your company.

## What Stage is Your Company In?

The tech field is home to companies of all shapes and sizes, from two guys in a basement to global corporations with hundreds of divisions. Where do you fall in the spectrum? The answer to that question has huge implications. It will shape how you structure your budget, what you do to try to draw new customers, and how you present your brand.

When it comes to choosing the right tech stack for your app, the size and positioning of your company is also of great importance. Let's take a look at the appropriate tech stack for companies of a few different sizes.

### Basement Bootstrappers

You've got a great idea and you're sure it's going to make you bags of cash. Maybe this is your first rodeo, maybe not. Either way, your goal is clear: get it built, and get it in front of people so that you can prove your concept.

For tech companies in this early stage, it's vital for adding features to be cheap and quick. You're almost certainly following an [MVP process](https://blog.engineyard.com/2015/actually-mvp), which means you're going to be throwing a lot of work out.

When you know that your code is disposable, you'll want to use the easiest tool you can find. Some good choices include [WordPress](https://wordpress.org/), basic PHP, or a static site generated with [Jekyll](https://jekyllrb.com/) or [HarpJS](http://harpjs.com/). All of these technologies are easy to understand, and you can probably get started with them yourself even if you're not especially technical.

In some cases, you might already know that you need to build something that will last a little while. If you've already proven your concept in a basic way, and need to set up a solid base that you can easily extend from, but you're still very young, you might want to consider using [Ruby on Rails](http://rubyonrails.org/) or [Django](https://www.djangoproject.com/). Both frameworks are easy to build new features in. Plus, they have the added bonus of being widely known, which means you will find it easier to hire new developers to work on your code as you continue to grow.

### Proven Market

If you've been in the game for a while, and you know you're on to something with your core offering, you'll want to approach your choice a little differently.

If you've made it to this point, there's a pretty good chance that you've done so via an easy-to-use technology. Perhaps WordPress or a static site. Maybe you find yourself needing to add new features at a quick pace, but the tech stack you've been using is proving cumbersome. It was great for building a basic interface, but setting up custom features requires you to stretch it beyond its intended purpose.

At this point, you may well make the choice to switch to a new framework. You should think about your strategy. Are you planning to add a few more core features and work on refining them, or are you going to be doing a lot of experimentation for the foreseeable future?

If you think you know where you're headed, and your main concern is scalability and stability, you might skip past flexible frameworks like Ruby on Rails and go straight for something beefier like [Go](https://golang.org/) or [Scala](http://www.scala-lang.org/). However, that can sometimes be a pretty big risk. You're basically betting that your business will stay mostly the same for the foreseeable future, because developing in the beefier languages takes more time and money. Still, picking one of these more "industrial strength" solutions is the way to go if you want to build something that will last.

On the other hand, if you want to keep playing with secondary features as you go, you might switch to something like Rails, or Django. These technologies will give you the opportunity to play around as much as you like. The risk you take in going this direction is that you will end up with a lot of spaghetti code, because these technologies are so flexible that you can say the same things in many ways, and your developers inevitably will do just that. If you continue down this path, maintenance will get more and more challenging as time goes on.

Of course, you may find that you don't need to make a switch yet at all. If your current stack is meeting your needs, stick with it! Delay switching as long as you reasonably can, because there's a pretty strong chance that [You Ain't Gonna Need It](http://martinfowler.com/bliki/Yagni.html)!

### Established and Growing

The next stepping stone of growth after you've proven your market and continued to grow it comes when you reach the point of being quite well-known and having tons of users. You probably have a code base that has a lot of technical debt, because the strain of doing what's needed to keep the business strong and growing usually necessitates making some sacrifices in the quality of your code.

At this point, your main concerns are paying off tech debt, making maintenance and development faster, and scalability under load.

This is the moment when most companies begin to consider the benefits of doing [the Rewrite](http://programmers.stackexchange.com/questions/6268/when-is-a-big-rewrite-the-answer). The rewrite is an expensive, multi-month process in which your product's existing functionality is coded from scratch with a new tech stack (and usually, UI), then switched out. It's "Your Company 2.0".

At this point, you need to decide on your organizational architecture. Will you go toward a micro-services or other [Service-Oriented Architecture](http://fluxusfrequency.github.io/blog/2014/02/14/service-oriented-architecture/)? If so, consider the difficulty of making sure that there is clear encapsulation and well-defined APIs for each of your services. Perhaps you want to stick with a [monolith](https://m.signalvnoise.com/the-majestic-monolith-29166d022228#.deg42umkz), but it's time to start over and apply the lessons you've learned. Or, maybe you want to move from a server-rendered page style app to a JavaScript heavy client-side app.

If you have an established and growing business and you find yourself facing this decision, you should definitely consult with an experienced system architect and weigh the benefits and drawbacks of each approach you're considering. Because the rewrite is an expensive and time-intensive process, it greatly weakens your ability to add new features while it's going on. Having the guidance of an skilled architect will help mitigate the cost and risk of making the switch-over.

### Evaluating Technologies

At this point, you've asked some basic questions about what kind of experience you want to provide your users, and you've identified what type of company you have. Hopefully this process has helped you generate a list of potential technologies. Before we conclude, let's take a moment to gauge each choice on its own merits.

Drawing from my colleague Laura Steadman's article on [Evaluating Open Source Libraries](https://quickleft.com/blog/evaluating-open-source-libraries-five-questions-to-ask/), I've created a list of questions to ask when you're looking at a tool that you might like to use.

- Is it well-documented?
- Is there an active community around it?
- If it's a new framework, how quickly is it changing?
- Does it have a corporate backer? If so, what is their track record in supporting technologies?
- Is it easy to test?
- How difficult will it be to hire developers to work on it?
- What does the ramp-up time for learning it look like?
- Is there something unique about your business needs that only this technology can provide?
- When it comes to hosting and DevOps, do you have resources available to support changes on your own, or does it make sense to pay for a Platform as a Service provider like [Heroku](https://www.heroku.com/) or [Engine Yard](https://engineyard.com/)?

## Conclusion

When you're piloting a tech business, there are certain touchstone moments where the decisions you make can make or break your future success. Choosing the right tech stack for your app is one of these moments. In this post, we've looked some strategies you can use when making this choice.

First, we thought about the demographics and experience you want to provide. Next, we considered where your business is in its overall lifecycle. Finally, we went down a checklist and asked questions to evaluate how appropriate each of your choices was.

If you're facing this decision, I hope this post has helped you face the question with a little more rationality and forethought. Best wishes for your business and success!

Until next time, happy hacking!
