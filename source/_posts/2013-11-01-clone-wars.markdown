---
layout: post
title: "Clone Wars"
date: 2013-11-01 09:15
---

This week, our assignment in gSchool was to rip an existing website from a local business, rebuild the site using Sinatra on the back end, and add a Content Managament System (CMS) so the site owner could login and edit the site. The project was called [Clone Wars](http://tutorials.jumpstartlab.com/projects/clone_wars.html). I was grouped with classmates Luke Martinez and Antony Siegert. We chose to rip [The Bike Depot](http://thebikedepot.org/). Since we never got our site pushed up to Heroku, but here is a screenshot of our CMS:

{%img http://s5.postimg.org/n4h93oltz/Screen_Shot_2013_11_01_at_9_20_51_AM.png 'The Bike Depot CMS' 'A website with a Content Management System' %}

##What I Learned

Although I already felt pretty comfortable with Sinatra and HTTP requests before we began Clone Wars, I also learned quite a bit along the way.

This was the first project where we were really able to use a database, so I got some good practice using the Sequel gem to talk to sqlite3. I posted a summary of Sequel methods on [a post](http://fluxusfrequency.github.io/blog/2013/10/26/sequel-gem-methods/) earlier this week. Clone Wars was a good chance to practice accessing the database through class objects, like this:

```ruby page_store.rb
require 'sequel'

class PageStore
  class << self

    def save(page)
      page_table.insert({
        "title" => page.title,
        "body" => page.body,
        "url" => page.url,
        })
    end

    def find(id)
      result = page_table.where(:id => id).to_a.last
      Page.new(
        {"id" => result[:id],
        "body" => result[:body],
        "title" => result[:title],
        "url" => result[:url] }
        )
    end

    def update(page, attributes)
      found = find_by_url(page.url)
      new_page = Page.new(found.to_h.merge(attributes))
      delete(found)
      save(new_page)
      new_page
    end

    def delete(page)
      page_table.where(:url => page.url).delete
    end

    def delete_all
      page_table.delete
    end


    def database
      if ENV['RACK_ENV'] == 'test'
        @database ||= Sequel.sqlite('./test.sqlite3')
      else
        @database ||= Sequel.sqlite('./clone_wars.sqlite3')
      end
    end

    def page_table
      database[:pages]
    end

  end
end

```

I also had a great time using the Mechanize gem to rip the html files from the original site. We split the head, header, navbar, and footer sections off into a layout file, and sticking a <%= yield %> tag in its body, then scraped the main content of each page on the site and rendered it as a partial.

```ruby scraper.rb
require 'nokogiri'
require 'open-uri'
require 'fileutils'

urls = [
'http://thebikedepot.org/',
'http://thebikedepot.org/index.php/about',
'http://thebikedepot.org/index.php/about/mission-vision-and-values'
# etc...
]

urls.each do |url|
  page = Nokogiri::HTML(open(url))
  body = page.css("div#content-w2")
  title = page.title.to_s
  FileUtils.touch("./scraped/#{title}.html")
  File.open("./scraped/#{title}.html", "w") do |file|
    file.write(body)
  end
end

```

Finally, I got some practice using the FileUtils library to hand write a database migration. If I had to do it again, I'd use a Rake task, but it was fun to hack it together this way. Given a directory of erb files, I collected their filenames, html bodies, and routes, combined them into hashes, and inserted them into the database with this:

```ruby database_populater.rb
require 'fileutils'
require 'sequel'

titles = []
pgs = []
urls = [
'/about',
'/programs/bike-camp',
'/programs/bike-rodeo',
'/bike-shop',
'/about/staff-board/17-about/staff-board/43-bill-davis',
'/about/staff-board/17-about/staff-board/7-board'
# etc...
]

database = Sequel.sqlite('clone_wars.sqlite3')

Dir["./lib/app/views/*.erb"].each do |file|
  filename = file.to_s.gsub(/.erb/, "")
  filename = filename.gsub(/\.\/lib\/app\/views\//, "")
  titles << filename
  pgs << File.open(file, "r").read
end


pgs.each_with_index do |page, i|
  hash = {:title => titles[i], :body => pgs[i], :url => urls[i]}
  database.from(:pages).insert(hash)
end

```

##Confusing Parts

One part of this project that was really confusing was an inital misstep I made: trying to convert all of the scraped html files into Ruby Slim. I mechanized the process of converting them with an [html-to-slim](http://html2slim.herokuapp.com/) conversion website, but there were way too many problems with the resulting files. I ended up wasting several hours going through this process before scrapping it.

##Acceptance Tests

I still feel unsure about acceptance tests and Capybara. Although I know how to get started, I have trouble using it for TDD. Here's what I started with:

```ruby acceptance_test.rb

require './test/helpers/acceptance_helper.rb'
require './lib/clone_wars'
require './lib/app'

class BikeCoopAppTest < Minitest::Test
  include Capybara::DSL

  def app
    @app ||= BikeCoopApp.new
  end

  def test_it_exists
    visit '/'
    assert page.has_content?("Bike Depot")
    assert page.has_content?("Home")
  end

  def test_it_has_a_login_link
    visit '/'
    click_on '#admin'
    assert page.has_content?("Login")
    assert page.has_content?("Username:")
  end

end

```

This was probably a good start, but I also feel like I have to test *everything*, and get overwhelmed. In this case, I didn't want to test all of the interactions from the scraped pages, because everything already fit together. When building someone else's site, Capybara seems superfluous. On the other hand, when I used it with a site I was building from scratch ([IdeaBox](http://ideabox-flux.herokuapp.com/)), I was annoyed that the test would break whenever I changed my mind about the layout. I spoke to Katrina Owen about it, and she suggested only using acceptance tests for parts of the site that you are going to have to test repeatedly anyway, rather than every single route.

I'm going to keep working on the practice of Capybara, because I believe deeply that TDD is the best way to go. This project was unsuccessful in this regard, but it's been a valuable learning experience, and will surely change the way I test my future projects.

##If I Had More Time...

If I had more time for this project, I would have build a better CMS that displayed the text to be edited in either markdown or something like [CK Editor](http://ckeditor.com/), rather than raw html, to provide a better user experience for the site administrator.

##Working In A Trio

Working in a three person group was much different than working solo or pairing. I really enjoyed having other minds working on the same problem, seeing solutions that I might not have considered. Both Luke and Antony were very positive about screensharing and mobbing. We practiced some 'dummy keyboard' and 'navigator' approaches, both of which worked really well. I also really enjoyed being able to split up and work on separate features of the project, because it was a load off my shoulders compared to IdeaBox, where I had to build all of the features myself. One thing that was challenging about working in a trio was being unable to move forward until a certain task was done. For example, we couldn't write any classes until we got the test environment set up correctly. It was also hard to slow down and explain my thinking at each step, rather than just blasting through everything. But this process was probably good for me. I tend to work through problems internally, and being able to process out loud will be a valuable soft skill to have down the road when I begin working as a professional developer.

##Final Thoughts

In retrospect, I wasn't a huge fan of the Clone Wars project. I would prefer to create something new, rather than take an existing layout and add a wrapper to it. However, I was pretty stimulated by some aspects of the project: using mechanize, sequel, and a full fledged database. I'm looking forward to starting Rails next week, and getting even deeper into working with ORM patterns and small groups.
