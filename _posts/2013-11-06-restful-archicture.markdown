---
title: RESTful Architecture
date: 2013-11-06 06:11:00 Z
categories:
- source
layout: post
---

The gSchool students began our first big Rails project this week. We're building a restaurant site with online ordering. The project is called [Dinner Dash](http://tutorials.jumpstartlab.com/projects/dinner_dash.html).

In support of learning Rails, Jeff gave a talk on how RESTful Archecture should look in a Rails application. Here are my notes on what he said. I broke it down into CRUD (Create, Read, Update, Destroy) operations.

I've shown the HTTP request verbs and routes and the controller methods that match them. It should be noted that GET will display request params in its URL, while POST requests will carry request params hidden in the body of the HTTP request.

For the examples, I used 'articles' as a resource. For single records, I referred to an article with an id of 1, although many different ids would actually exist. I also assumed that strong params are defined in a method called 'article_params' in the Articles Controller. These patterns came from the Jumpstart Lab [Blogger Tutorial](tutorials.jumpstartlab.com/projects/blogger.html).

<br/>

## Setting Up The Router

In config/routes.rb:

```ruby
resources :articles
```

<br/>

##Create

### Creating A Single Record

- GET '/articles/new' (displays the new article form)
- POST '/articles/new'

In app/controllers/articles_controller.rb:

```ruby
def create
  @article = Article.new(article_params)
  @article.save
end
```

<br />

## Read

### Reading Multiple Records

- GET '/articles'

```ruby
def index
  @articles = Article.all
end
```

### Reading A Single Record

- GET '/articles/1'

```ruby
def show
  @article = Article.find(params[:id])
end
```

<br />

## Update

### Updating A Single Record

- GET '/articles/1/edit' (displays the edit form)
- PUT '/articles/1'

```ruby
def update
  @article = Article.find(params[:id])
  @article.update(article_params)
end
```

<br />

## Destroy

### Deleting A Single Record

- DELETE '/articles/1'

```ruby
def destroy
  @article = Article.find(params[:id])
  @article.destroy
end
```

<br/>

## Search

### Searching With A Filter

- GET '/articles?author_id=6'

### Saved Searches

- POST '/searches' (creates the search)
- GET '/searches/12' (accesses the saved search)

