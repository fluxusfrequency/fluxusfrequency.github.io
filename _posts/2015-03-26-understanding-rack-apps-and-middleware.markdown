---
title: Understanding Rack Apps and Middleware
date: 2015-03-26 12:29:45 Z
categories:
- source
layout: post
comments: true
---

This post originally appeared on [Engine Yard](https://blog.engineyard.com/2015/understanding-rack-apps-and-middleware).

## Introduction

For many of us web developers, we work on the highest levels of abstraction when we program. Sometimes it's easy to take things for granted. Especially when we're using Rails.

Have you ever dug into the internals of how the request/response cycle works in Rails? I recently realized that I knew almost nothing about how Rack or middlewares work, so I spent a little time finding out. In this post, I'll share what I learned.

## What's Rack?

Did you know that Rails is a [Rack](http://rack.github.io/) app? Sinatra too. What is Rack? I'm glad you asked. Rack is a Ruby package that provides an easy-to-use interface to the Ruby [Net::HTTP](http://ruby-doc.org/stdlib-2.2.0/libdoc/net/http/rdoc/Net/HTTP.html) library.

It's possible to quickly build simple web applications using just Rack.

To get started, all you need is an object that responds to a `call` method, taking in an environment hash and returning an Array with the HTTP response code, headers, and response body. Once you've written the server code, all you have to do is boot it up with a Ruby server like `Rack::Handler::WEBrick`, or put it into a `config.ru` file and run it from the command line with `rackup config.ru`.

Ok, cool. So what does Rack actually _do_?

## How Rack Works

Rack is really just a way for a developer to create a server application while avoiding the boilerplate code that would be required to do so using `Net::HTTP`. If you've written some code that meets the Rack specifications, you can load it up in a Ruby server like `WEBrick`, `Mongrel`, or `Thin`, and you're ready to accept requests and respond to them.

There are a few methods you should know about that are provided for you. You can call these directly from within your `config.ru` file.

***`run`***
Takes an application (the object that responds to `call`) as an argument. The following code from the Rack website demonstrates how this looks:

```
run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['get rack\'d']] }
```

***`map`***
Takes a string specifying the path to be handled, and a block containing the Rack application code to be run when a request with that path is received. Here's an example:

```
map '/posts' do
  run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['first_post', 'second_post', 'third_post']] }
end
```

***`use`***
Tells Rack to use certain middleware.

So what else do you need to know? Let's take a closer look at the environment hash and the response Array.

### The Environment Hash

Your Rack server object takes in an environment hash. What's contained in that hash? Here are a few of the more interesting parts:

- `REQUEST_METHOD`: The HTTP verb of the request. This is required.
- `PATH_INFO`: The request URL path, relative to the root of the application.
- `QUERY_STRING`: Anything that followed `?` in the request URL string.
- `SERVER_NAME` and `SERVER_PORT`: The server's address and port.
- `rack.version`: The rack version in use.
- `rack.url_scheme`: is it `http` or `https`?
- `rack.input`: an IO-like object that contains the raw HTTP POST data.
- `rack.errors`: an object that response to `puts`, `write`, and `flush`.
- `rack.session`: A key value store for storing request session data.
- `rack.logger`: An object that can log interfaces. It should implement `info`, `debug`, `warn`, `error`, and `fatal` methods.

A lot of frameworks built on Rack wrap the `env` hash in a `Rack::Request` object. This object provides a lot of convenience methods. For example, `request_method`, `query_string`, `session`, and `logger` return the values from the keys described above. It also lets you check out things like the `params`, HTTP `scheme`, or whether you're using `ssl?`. For a complete listing of methods, I would suggest digging through [the source](https://github.com/rack/rack/blob/master/lib/rack/request.rb).

### The Response

When your Rack server object returns a response, it must contain three parts: the status, headers, and body. As there was for the request, there is a [Rack::Response](https://github.com/rack/rack/blob/master/lib/rack/response.rb) object that gives you convenience methods like `write`, `set_cookie`, `finish`, and more. Alternately, you can just return an array containing the three components.

#### Status

An [HTTP status](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html), like 200 or 404.

#### Headers

Something that responds to each, and yields key-value pairs. The keys have to be strings and conform to the [RFC7230 token specification](https://tools.ietf.org/html/rfc7230). Here's where you can set `Content-Type` and `Content-Length` if it's appropriate for your response.

#### Body

The body is the data that the server sends back to the requester. It has to respond to `each`, and yield string values.

### All Racked Up!

Now that we've created a Rack app, how can we customize it to make it actually useful? The first step is to consider adding some middleware.

## What is Middleware?

One of the things that makes Rack so great is how easy it is to add a chain middleware components between the webserver and the app to customize the way your request/response behaves. But what _is_ a middleware component?

A middleware component sits between the client and the server, processing inbound requests and outbound responses. Why would you want to do that? There are tons of [middleware components available for Rack](https://github.com/rack/rack/wiki/List-of-Middleware) that take the guesswork out of problems like enabling caching, authentication, trapping spam, and many other problems.

## Using Middleware in a Rack App

To add middleware to a Rack application, all you have to do is tell Rack to use it. You can use multiple middleware components, and they will change the request or response before passing it on to the next component. This series of components is called the _middleware stack_.

### Warden

We're going to take a look at how you would add [Warden](https://github.com/hassox/warden) to a project. Warden has to come _after_ some kind of session middleware in the stack, so we'll use `Rack::Session::Cookie` as well.

First, add it to your project `Gemfile` with `gem "warden"` and install it with `bundle install`.

Now add it to your `config.ru` file:

```
require "warden"

use Rack::Session::Cookie, secret: "MY_SECRET"

failure_app = Proc.new { |env| ['401', {'Content-Type' => 'text/html'}, ["UNAUTHORIZED"]] }

use Warden::Manager do |manager|
  manager.default_strategies :password, :basic
  manager.failure_app = failure_app
end

run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['get rack\'d']] }
```

Finally, run the server with `rackup`. It will find `config.ru` and boot up on port 9292.

Note that there is more setup involved in getting Warden to actually do authentication with your app. This is just an example of how to get it loaded into the middleware stack. To see a more fleshed-out example of integrating Warden, check out [this gist](https://gist.github.com/lukesutton/107966).

By the way, there's another way to define the middleware stack. Instead of calling `use` directly in `config.ru`, you can use `Rack::Builder` to wrap several middlewares and app(s) in one big application. For example:

```
failure_app = Proc.new { |env| ['401', {'Content-Type' => 'text/html'}, ["UNAUTHORIZED"]] }

app = Rack::Builder.new do
  use Rack::Session::Cookie, secret: "MY_SECRET"

  use Warden::Manager do |manager|
    manager.default_strategies :password, :basic
    manager.failure_app = failure_app
  end
end

run app
```

### Rack Basic Auth

One really useful piece of middleware is `Rack::Auth::Basic`, which you can use to protect any Rack app with [HTTP basic authentication](http://tools.ietf.org/html/rfc2617). It is really lightweight and comes in handy for protecting little bits of an application. For example, Ryan Bates uses it to protect a Resque server in a Rails app in [this episode of Railscasts](http://railscasts.com/episodes/271-resque?view=asciicast).

Here's how to set it up:

```
use Rack::Auth::Basic, "Restricted Area" do |username, password|
  [username, password] == ['admin', 'abc123']
end
```

That was easy!

## Using Middleware in Rails

Now, so what? Rack is pretty cool, and we know that Rails is built on it. But just because we understand what it is, doesn't make it actually useful in working with a production app.

### How Rails Uses Rack

Did you ever notice that there's a `config.ru` file in the root of every generated Rails project. Have you ever taken a look inside? Here's what it contains:

```
# This file is used by Rack-based servers to start the application.

require ::File.expand_path('../config/environment', __FILE__)
run Rails.application
```

Pretty simple. It just loads up the `config/environment` file, then boots up `Rails.application`. Wait, what's that? Taking a look in `config/environment`, we can see that it's defined in `config/application.rb`. `config/environment` is just calling `initialize!` on it.

So what's in `config/application.rb`? If we take a look, we see that it loads in the bundled gems from `config/boot.rb`, requires `rails/all`, loads up the environment (test, development, production, etc.), and defines a namespaced version of our application. It looks something like this:

```
module MyApplication
  class Application < Rails::Application
    ...
  end
end
```

So I guess that means that `Rails::Application` must be a Rack app? Sure enough! If we check out [the source code](https://github.com/rails/rails/blob/master/railties/lib/rails/application.rb#L161), it responds to `call`!

So what middleware is it using? Well, I see that [it's autoloading](https://github.com/rails/rails/blob/master/railties/lib/rails/application.rb#L182) `rails/application/default_middleware_stack`. [Checking that out](https://github.com/rails/rails/blob/master/railties/lib/rails/application/default_middleware_stack.rb#L13), it looks like it's defined in `ActionDispatch`. Where does `ActionDispatch` come from? `ActionPack`.

### Action Dispatch

[Action Pack](https://github.com/rails/rails/tree/master/actionpack) is Rails's framework for handling web requests and responses. Action Pack home to quite a few of the niceties you find in Rails, such as routing, the the abstract controllers that you inherit from, and view rendering.

The most relevant part of AP for our discussion here is [Action Dispatch](https://github.com/rails/rails/tree/master/actionpack/lib/action_dispatch). It provides several middleware components that deal with ssl, cookies, debugging, static files, and much more.

If you go take a look at each of the [Action Dispatch middleware components](https://github.com/rails/rails/tree/master/actionpack/lib/action_dispatch/middleware), you'll notice they're all following the Rack specification: they all respond to `call`, taking in an `app` and returning `status`, `headers`, and `body`. Many of them also make use of `Rack::Request` and `Rack::Response` objects.

For me, reading through the code in these components took a lot of the mystery out of what's going on behind the scenes when making requests to a Rails app. When I realized that it's just a bunch of Ruby objects that follow the Rack specification, passing the request and response to each other, it made this whole section of Rails a lot less mysterious.

Now that we understand a little bit of what's happening under the hood, let's take a look at how to actually include some custom middleware in a Rails app.

### Adding Your Own Middleware

Imagine you are [hosting an application on Engine Yard](https://www.engineyard.com/trial). You have a Rails API running on one server, and a client-side JavaScript app running on another. The API has a url of `https://api.myawesomeapp.com`, and the client-side app lives at `https://app.myawesomeapp.com`.

You're going to run into a problem pretty quick: you can't access resources at `api.myawesomeapp.com` from your JS app, because of the [same-origin policy](http://en.wikipedia.org/wiki/Same-origin_policy). As you may know, the solution to this problem is to enable [Cross-origin resource sharing](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (CORS). There are many ways to enable CORS on your server, but one of the easiest is to use the [Rack::Cors middleware gem](https://github.com/cyu/rack-cors).

Begin by requiring it in the `Gemfile`:

```ruby
gem "rack-cors", require: "rack/cors"

```

As with so many things, Rails provides a very easy way to get middleware loaded. Although we certainly _could_ add it to a `Rack::Builder` block in `config.ru`, as we did above, the Rails convention is to place it in `config/application.rb`, using the following syntax:

```ruby
module MyAwesomeApp
  class Application < Rails::Application
    config.middleware.insert_before 0, "Rack::Cors" do
      allow do
        origins '*'
        resource '*',
        :headers => :any,
        :expose => ['X-User-Authentication-Token', 'X-User-Id'],
        :methods => [:get, :post, :options, :patch, :delete]
      end
    end
  end
end
```

Note that we're using [insert_before](http://guides.rubyonrails.org/rails_on_rack.html#configuring-middleware-stack) here to ensure that `Rack::Cors` comes before the rest of the middleware included in the stack by ActionPack (and any other middleware you might be using).

Now if you reboot the server, you should be good to go! Your client-side app can access `api.myawesomeapp.com` without running into same-origin policy JS errors.

If you want to learn more about how HTTP requests are routed through Rack in Rails, I’d suggest taking a look at [this tour](https://www.omniref.com/ruby/gems/railties/4.2.0/symbols/Rails::Application#annotation=4084035&line=161) of the Rails source code that deals with handling requests.

## Conclusion

In this post, we've take an in-depth at the internals of Rack, and by extension, the request/response cycle for several Ruby web frameworks, including Ruby on Rails.

Hopefully, understanding what's going on when a request hits your server and your application sends back a response helps make things feel a little less _magical_. Because I don't know about you, but when things go wrong, I have a lot harder time troubleshooting when there's magic involved than when I understand what's going on. In that case, I can say "oh, it's just a Rack response", and get down to fixing the bug.

If I've done my job, reading this article will enable you to do the same thing.

P.S. Do you know of any use-cases where a simple Rack app was enough to meet your business needs? What other ways do you integrate Rack apps in your bigger applications? We want to hear your battle stories! Leave us a comment!


