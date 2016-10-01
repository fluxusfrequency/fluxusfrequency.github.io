---
title: Sales Engine Retrospective
date: 2013-10-04 09:08:00 Z
categories:
- source
layout: post
---

We've spent the last week and a half at gSchool working on the next hardest thing to EventReporter (from week two): [SalesEngine](http://tutorials.jumpstartlab.com/projects/sales_engine.html).

## The Code and The Problem

This project built on our skills of creating collections of objects, and building classes that inherit multiple attributes from CSV rows. This time, instead of one class, we had six: Customers, Invoices, InvoiceItems, Items, Merchants, and Transactions. Using six corresponding CSV files, we had to build basic class definitions for each and repositories to hold data about the collections found in the CSV files.

It was easy enough to stitch those together in a basic way, but the real challenge was finding ways to implement the "business intelligence" parts of the assignment. In these places, classes had to reach across many other classes to access data. It was very difficult to make this work correctly. Jeff told us there was really no good solution to the problem using pure Ruby. You either have to create a database class that ends up being a global variable, or pass information between many classes, creating high coupling. I assume that we're being asked to feel this pain so we will be very thankful when we're finally introduced to databases and/or ActiveRecord. I was really hurting for an SQL INNER JOIN.

Here's an example of one of the problems we faced:

The ItemRepository needs to have a method called most_revenue(x) that returns the top x Item instances ranked by total revenue generated. But the Item itself doesn't have an attribute for revenue. You have to access that through the InvoiceItem class, which must be accessed via an Invoice. Therefore, for the ItemRepository to find out the Item with the most revenue, it has to talk to the Invoice, which can then find the InvoiceItems.

Reaching across classes like this is very difficult, and the process of figuring it out was a good lesson in following the [Law of Demeter](http://devblog.avdi.org/2011/07/05/demeter-its-not-just-a-good-idea-its-the-law/).

It was really tempting to write code like this:

```ruby ItemRepository#most_revenue(x) - Bad Implementation
class SalesEngine
  class ItemRepository

  # some code omitted

  def most_revenue(x)
    sum = BigDecimal.new(0)
    result = items.collect do |item|
      item.invoice.invoice_items.each do |invoice_item|
        sum += BigDecimal.new(invoice_item.total)
      end
    end

    result.flatten.sort.reverse
  end

  # some more code omitted

  end
end

```

The problem with this code is that it has to iterate through all of the Items (there are 2,484 of them), and iterate through each of their Invoices, then each Invoice's InvoiceItems, totalling each one, to get the result. A better solution is to leave the lookups to each class, never going more than one level deep to get information. It also helps to use the Enumerable#group_by to store the total for each item in a hash for a faster lookup. Here's what that looks like:

```ruby

{5243 => #<Merchant 1>, 1523 => #<Merchant 2>, etc...}

```

You can then easily lookup the Merchant with the greatest revenue.
Here was my eventual solution to this problem, using #group_by:

```ruby ItemRepository#most_revenue(x) - Better Implementation
class SalesEngine
  class ItemRepository

  # some code omitted

  def most_revenue(x)
    sorted_totals = items_grouped_by_revenue.keys.sort.reverse

    most_revenue = sorted_totals.collect do |total|
      items_grouped_by_revenue[total]
    end

    most_revenue.flatten[0,x]
  end

  def items_grouped_by_revenue
    items.group_by { |item| item.revenue }
  end

  # some more code omitted

  end
end

class SalesEngine
  class Item

  # some code omitted

  def revenue
    total_up(successful_invoice_items)
  end

  def total_up(invoice_items)
    sum = invoice_items.collect { |invoice_item| invoice_item.total }.inject(0,:+)
    BigDecimal.new(sum)/100
  end

  # some code omitted

  end
end

```

## Pairing

SalesEngine was meant to be a pairing project. I started off pairing with my classmate Louisa Barrett. Things started off well. We created a skeleton of the project, then split up the work of building the basic classes. We did the same with the tests, the repositories and the repository tests. But when it came time to implement searching and business intelligence methods, Louisa started having trouble. She said she "hit a wall". At this point, she paired with another classmate who was feeling the same way, and I went on by myself.

I think the difference between us was that she is a novice and I'm an advanced beginner, labels from Andy Hunt’s [Pragmatic Thinking & Learning](http://pragprog.com/book/ahptl/pragmatic-thinking-and-learning). Our teacher, Katrina Owen, recently explored how these levels of learning are playing out at gSchool on the [Jumpstart Lab blog](http://jumpstartlab.com/news/archives/2013/10/03/pragmatic-learning-at-gschool-part-i).

I don't mind working alone. I do well with challenges on my own. It's also easier when you don't have to compromise your ideas of what should come next. But sometimes, I did feel a little jealous of some of my advanced beginner classmates that were paired together. It looked like they were learning a lot from each other. Hopefully, I'll get to pair with someone on the next project.

## The Rebuild

The day before the project was due, I was starting to hit a wall. I'd finished implementing all the features, so I ran the spec harness. There were over thirty failures. Franklin looked at my code and rewrote huge chunks of it. I started to lose confidence in my archtecture.

At 6:00 the night before it was due, I decided to tear it down and rebuild. It actually went pretty quickly, now that I knew what everything would look like. I built up all the classes and repos again. I recreated the SalesEngine class and the Database. I was practicing TDD with MiniTest all the way. I ran the spec with the business intelligence tests commented out. Everything passed, except a few places where I had typos. I fixed them, and everything in my tests and the spec was green.

My next step was to take all of the specs for business intelligence, and rewrite them as an integration test in Minitest. I rewrote all the tests and skipped them, then started passing them one at a time. But there was a problem. It was running *really* slow, because it was loading up all six CSV files before each test (in the setup method). I wanted a way to do "before all" like you can in Rspec, but was having trouble finding it. Finally I found a hack online using class variables. Katrina says there's a better way, that she'll show me soon. In the meantime, here's what I did:

```ruby Integration Test - "Before All" Hack
gem 'minitest'
require 'minitest/autorun'
require 'minitest/pride'
require_relative '../../lib/sales_engine.rb'

class BusinessIntelligenceTest < Minitest::Test

  def self.before_suite
    SalesEngine::Database.startup('./data')
    @@database = SalesEngine::Database
    @@customer_repository ||= @@database.customer_repository
    @@invoice_repository ||= @@database.invoice_repository
    @@invoice_item_repository ||= @@database.invoice_item_repository
    @@item_repository ||= @@database.item_repository
    @@merchant_repository ||= @@database.merchant_repository
    @@transaction_repository ||= @@database.transaction_repository
  end

  before_suite

  def test_the_customer_transactions_method_successfully_returns_transactions
    customer = @@customer_repository.find_by_id 2
    assert_equal 1, customer.transactions.length
    assert_equal SalesEngine::Transaction, customer.transactions.first.class
  end

  def test_the_customer_favorite_merchant_method_successfully_returns_a_merchant
    skip
    customer = @@customer_repository.find_by_id 2
    assert_equal "Shields, Hirthe and Smith", customer.favorite_merchant.name
  end

  # many tests omitted

end

```
This approach really paid off. I passed all of the spec tests one at a time, before finally unleashing the spec on it. In the end, there were still two tests failing. One of them was a problem with the spec itself. The other I haven't solved yet, but will in the next couple of days.

In the end, I rebuilt the whole project between 6PM and noon the following day. I think it was a really great exercise to start over. I understood everything I was doing the second time through, and did it without reaching through the classes. One thing I really appreciated was that if you're going to run a spec harness, you might as well write your app around it from the start.

In all, the second time around went quickly because I knew what I was building. I also solidified a lot of the things I'd learned by repeating them. I'm glad that I did it.

## gSchool Progress

We're now four weeks into the program at gSchool. I've mastered more than I would have predicted. Well, maybe not quite 'mastered'. I do feel like I have a firm grasp of some things in Ruby: control flow, collections, classes, methods, and where to find help. Other things I understand, and can use somewhat, but don't have full mastery of: enumerables, procs, lambdas, and throwing exceptions.

## The Future

In five months, I believe I'll feel fully confident in Ruby skills. I'm really looking forward to digging into Rack, Sinatra, Rails, APIs, and front end skills in the coming weeks. Using Ruby in the context of building web apps is going to awesome. Also, now that I understand how programming languages work, I'll probably sample some other languages. Ruby is my focus right now, but I can definitely see myself dipping my toes into Javascript, Go, Python, and maybe some C down the line.

When gSchool is over, I'm most interested in getting a job for a consultancy. At this point in my development, I have my eyes on companies like Thoughtbot and Pivotal Labs, where the pursuit of programming as a discipline is a central tenet of the company culture. I'm curious to see if my ideas about where I would fit in the industry will change as I learn more about it between now and March.

## My Learning Style

So far in this program, I've spent a lot of time learning on my own. Twice in a row, I've lost my pair halfway through a project (was it something I said?). But that doesn't bother me. After college, I made it a point to keep teaching myself, so I've also become quite adept at self-directed learning. Some of the stragies I use that have been working well at gSchool include: sketching out the big picture before digging in on a project, using the Ruby documentation extensively, asking for help from teachers or advanced classmates when I need it, and watching my Twitter feed/trolling the web for interesting GitHub repos, screencasts, and tutorials that will improve my understanding and workflow.

The community in gSchool is really awesome; I feel like I'm really in my element here. I got a little taste of pairing with Rolen Le early on, but haven't had many other chances to screenshare since then. I'd really like to do it more in the future. I'll bet that watching and hearing someone else think through problems will help me see approaches I wouldn't have come up with on my own. Eventually, I hope I have a chance to screenshare with all of my classmates, especially those that have more experience than me.

One thing that has been sometimes difficult are full class sessions covering a fairly basics topic that I already feel comfortable with. In these cases, I've found it hard to maintain focus. I feel that I would probably get more out of splitting off from the class and researching topics on my own during these sessions.

## Wrap-up

Now that SalesEngine has been turned in, it will be tempting to move on to the next thing and not look back. However, after doing the rebuild, I still have a couple of tests failing, and I want them green. The spec is still taking me a really long time to run, so I want to finish implementing the faster hash lookups that Franklin Webber showed us in the code review (see the code example above). I'll also probably run Cane and Reek to double check that my code is well-formatted. As I was telling another student, a project is never really 'done'. There are always more things that could be done to improve or extend it. After tying up these loose ends, I'll probably move on to the next project. Bootcamp is fast paced. But I'll also be interested to go back and read my code five months from now.

In Katrina's blog post, she described the first few tutorials that we did as a "guided tour of the universe". SalesEngine was our chance to find out how to pull all of the stars and planets together to create a galaxy. I felt like I was stretched in a *really* good way in this project, having to put all of the abstractions of pure Ruby together into something that made sense. If I look through the table of contents of Eloquent Ruby, I feel like I understand what all of the expressions, objects, and classes *are*. Many of them are tools I feel ready to use at any time, others I have at least touched. Now that I have seen each structure used in context, it has a much deeper meaning. Thanks to this project, the universe that is Ruby language makes a lot more sense to me.
