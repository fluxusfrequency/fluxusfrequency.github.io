---
title: Serving Custom JSON From Your Rails API With ActiveModel::Serializers
date: 2015-03-01 13:23:01 Z
categories:
- source
layout: post
comments: true
---

This post originally appeared on [Engine Yard](https://blog.engineyard.com/2015/active-model-serializers).

# Introduction

These days, there are so many different choices when it comes to serving data from an API. You can build it in Node with [ExpressJS](http://expressjs.com/), in Go with [Martini](http://martini.codegangsta.io/), Clojure with [Compojure](https://github.com/weavejester/compojure), and many more. But in many cases, you just want to bring something to market as fast as you can. For those times, I still reach for [Ruby on Rails](http://rubyonrails.org/).

With Rails, you can spin up a function API server in a very short period of time. Rails is large. Perhaps you object that there's "too much magic". Have you ever checked out the [rails-api gem](https://github.com/rails-api/rails-api)? It lets you enjoy all the benefits of Rails without including unnecessary view-layer and asset-related code.

Rails-api is maintained by [Carlos Antonio Da Silva](https://github.com/carlosantoniodasilva), [Santiago Pastorino](https://github.com/spastorino), Rails Core team members, and all-around great Rubyist [Steve Klabnik](http://www.steveklabnik.com/). While not busy working on Rails or the Rails API Gem, they found the time to put together the [active_model_serializers gem](https://github.com/rails-api/active_model_serializers) to make it easier to format JSON responses when using Rails as an API server.

ActiveModel::Serializers (AMS) is a powerful alternative to [jbuilder](https://github.com/rails/jbuilder), [rabl](https://github.com/nesquena/rabl), and other Ruby templating solutions. It's easy to get started with, but when you want to serve data that quite doesn't match up with the way ActiveRecord (AR) structures things, it can be hard to figure out how to get it to do what you want.

In this post, we'll take a look at how to extend AMS to serve up custom data in the context of a Rails-based chat app.

## Kicking It Off: Setting Up a Rails Server

Any two users in the system can have a continuous thread that goes back and forth. Let's imagine we are building a chat app, similar to Apple's Messages. People can sign up for the service, then chat with their friends.

Most of the presentation logic will happen in a client-side JavaScript app. For now, we're only concerned with accepting and returning raw data, and we've decided to use a Rails server to build it.

To get started, we'll run a `rails new`, but since we're using the `rails-api` gem, we'll need to make sure we have it installed first, with:

```
gem install rails-api
```

Once that's done, we'll run the following (familiar) command to start the project:

```
rails-api new mensajes --database=postgresql
```

`cd` into the directory and setup the database with:

```
rake db:create
```

## Creating the Models

We'll need a couple of models: `User`s and `Messages`. The workflow to create them should be fairly familiar:

```
rails g scaffold user username:string
rails g scaffold message sender_id:integer recipient_id:integer body:text
```

Open up the migrations, and set everything to `null: false`, then run `rake db:migrate`.

We'll also need to set up the relationships. Be sure to test these relationships (I would suggest using the [shoulda gem](https://github.com/thoughtbot/shoulda-matchers) to make it easy on yourself.

```
class User < ActiveRecord::Base
  has_many :sent_messages, class_name: "Message", foreign_key: "sender_id"
  has_many :received_messages, class_name: "Message", foreign_key: "recipient_id"
end
```

```
class Message < ActiveRecord::Base
  belongs_to :recipient, class_name: "User", inverse_of: :received_messages
  belongs_to :sender, class_name: "User", inverse_of: :sent_messages
end
```

## Serving the Messages

Let's send some messages! Imagine for a minute that you've already set up some kind of token-based authentication system, and you have some way of getting ahold of the user that is making requests to your API.

We can open up the `MessagesController`, and since we used a `scaffold`, we should already be able to view all the messages. Let's scope that to the current user. First we'll write a convenience method to get all the sent and received messages for a user, then we'll rework the `MessagesController` to work the way we want it to.

```
class User < ActiveRecord::Base
  ...
  def messages
    Message.where("sender_id = ? OR recipient_id = ?", self.id, self.id)
  end
end
```

```
class MessagesController < ApplicationController
  def index
    @messages = current_user.messages
    render json: @messages
  end
end
```

Assuming that we have created a couple of sent and received messages for the `current_user`, we should be able to take a look at `http://localhost:3000/messages` and see some raw JSON that looks like this:

```
[{"sender_id":1,"id":1,"recipient_id":2,"body":"YOLO","created_at":"2015-02-03T21:05:12.908Z","updated_at":"2015-02-03T21:05:12.908Z"},{"recipient_id":1,"id":2,"sender_id":2,"body":"Hello, world!","created_at":"2015-02-03T21:05:51.309Z","updated_at":"2015-02-03T21:05:51.309Z"}]
```

It's kind of ugly. It would be nice if we could remove the timestamps and ids. This is where AMS comes in.

## Adding ActiveModel::Serializers

Once we add AMS to our project, it should be easy to get a much prettier JSON format back from our `MessagesController`.

To get AMS, add it to the `Gemfile` with:

```
gem "active_model_serializers", github: "rails-api/active_model_serializers"

```

Then `bundle install`. Note that I'm using a the edge version of AMS here because it supports `belongs_to` and other features. See the [github project README](https://github.com/rails-api/active_model_serializers/blob/master/README.md) for some information about maintenance and why you might want to use an older version.

Now we can easily set up a serializer with `rails g serializer message`. Let's take a look at what this generated for us. In `app/serializers/message_serializer.rb`, we find this code:

```
class MessageSerializer < ActiveModel::Serializer
  attributes :id
end
```

Whichever `attributes` we specify (as a list of symbols) will be returned in the JSON response. Let's skip `id`, and instead return the `sender_id`, `recipient_id`, and `body`:


```
class MessageSerializer < ActiveModel::Serializer
  attributes :sender_id, :recipient_id, :body
end
```

Now when we visit `/messages`, we get this slightly cleaner JSON:

```
{"messages":[{"sender_id":1,"recipient_id":2,"body":"YOLO"},{"sender_id":2,"recipient_id":1,"body":"Hello, world!"}]}
```

## Cleaning Up the Format

It sure would be nice if we could get more information about the other user, like their username, so that we could display it in the messaging UI on the client-side. That's easy enough, we just change the `MessageSerializer` to use AR objects as attributes for the `sender` and `recipient`, instead of `id`s.

```
class MessageSerializer < ActiveModel::Serializer
  attributes :sender, :recipient, :body
end
```

Now we can see more about the Sender and Recipient:

```
{"messages":[{"sender":{"id":1,"username":"Ben","created_at":"2015-02-03T21:04:09.220Z","updated_at":"2015-02-03T21:04:09.220Z"},"recipient":{"id":2,"username":"David","created_at":"2015-02-03T21:04:45.948Z","updated_at":"2015-02-03T21:04:45.948Z"},"body":"YOLO"},{"sender":{"id":2,"username":"David","created_at":"2015-02-03T21:04:45.948Z","updated_at":"2015-02-03T21:04:45.948Z"},"recipient":{"id":1,"username":"Ben","created_at":"2015-02-03T21:04:09.220Z","updated_at":"2015-02-03T21:04:09.220Z"},"body":"Hello, world!"}]}
```

Actually, that might be too much. Let's clean up how `User` objects are serialized by generating a User serializer with `rails g serializer user`. We'll set it up to just return the username.

```
class UserSerializer < ActiveModel::Serializer
  attributes :username
end
```

In the `MessageSerializer`, we'll use `belongs_to` to have AMS format our `sender` and `recipient` using the `UserSerializer`:

```
class MessageSerializer < ActiveModel::Serializer
  attributes :body
  belongs_to :sender
  belongs_to :recipient
end
```

If we take a look at `/messages`, we now see:

```
[{"recipient":{"username":"David"},"body":"YOLO","sender":{"username":"Ben"}},{"recipient":{"username":"Ben"},"body":"Hello, world!","sender":{"username":"David"}}]
```

Things are really starting to come together!

## Conversations

Although we can view all of a user's messages using the `index` controller action, or a specific message at the `show` action, there's something important to the business logic of our app that we can't do. We can't view all of the messages sent between two users. We need some concept of a `conversation`.

When thinking about creating a conversation, we have to ask, does this model need to be stored in the database? I think the answer is no. We already have messages that know which users they belong to. All we really need is a way to get back all the messages between two users from one endpoint.

We can use a Plain Old Ruby Object (PORO) to create this concept of a `conversation` model. We will _not_ inherit from `ActiveRecord::Base` in this case.

Since we already know about the `current_user`, we really only need it to keep track of the other user. We'll call her the `participant`.

```
# app/models/conversation.rb
class Conversation
  attr_reader :participant, :messages

  def initialize(attributes)
    @participant = attributes[:participant]
    @messages = attributes[:messages]
  end
end
```

We'll want to be able to serve up these conversations, so we'll need a `ConversationsController`. We want to get all of the conversations for a given user, so we'll add a class-level method to the `Conversation` model to find them and return them in this format:

```
# TODO: Insert JSON blob here
```

To make this work, we'll run a `group_by` on the user's messages, grouping by the _other_ user's id. We'll then map the resulting hash into a collection of `Conversation` objects, passing in the other user and the list of messages.

```
class Conversation
  ...
  def self.for_user(user)
    user.messages.group_by { |message|
      if message.sender == user
        message.recipient_id
      else
        message.sender_id
      end
    }.map do |user_id, messages|
      Conversation.new({
        participant: User.find(user_id),
        messages: messages
      })
    end
  end
end
```

If we run this in the Rails Console, it seems to be working.

```
>Conversation.for_user(User.first)
...
=> [#<Conversation:0x007fbd6e5b9428 @participant=#<User id: 2, username: "David", created_at: "2015-02-03 21:04:45", updated_at: "2015-02-03 21:04:45">, @messages=[#<Message id: 1, sender_id: 1, recipient_id: 2, body: "YOLO", created_at: "2015-02-03 21:05:12", updated_at: "2015-02-03 21:05:12">, #<Message id: 2, sender_id: 2, recipient_id: 1, body: "Hello, world!", created_at: "2015-02-03 21:05:51", updated_at: "2015-02-03 21:05:51">]>]
```

Great! We'll just call this method in our `ConversationsController` and everything will be great!

First, we'll define the route in `config/routes.rb`:

```
Rails.application.routes.draw do
  ...
  resources :conversations, only: [:index]
end
```

Then, we'll write the controller action.

```
# app/controllers/conversations_controller.rb

class ConversationsController < ApplicationController
  def index
    conversations = Conversation.for_user(current_user)
    render json: conversations
  end
end
```

Visiting `/conversations`, we should see a list of all the conversations for the current user.

## Serializing Plain Old Ruby Objects

Whoops! When we visit that route, we get an error: `undefined method `new' for nil:NilClass`. It's coming from this line in the controller:

```
render json: conversations
```

It looks like the error is coming from the fact that we don't have a serializer. Let's make one with `rails g serializer conversation`. We'll edit it to return its attributes, `participant` and `message`.

```
class ConversationSerializer < ActiveModel::Serializer
  attributes :participant, :messages
end
```

Now when we try, we get another error, coming from the same line of the controller: `undefined method 'read_attribute_for_serialization' for #<Conversation:0x007ffc9c1bed10>`

Digging around in the source code for ActiveModel::Serializers, I couldn't find where that method was defined. So I took a look at ActiveModel itself, and found it [here](https://github.com/rails/rails/blob/08754f12e65a9ec79633a605e986d0f1ffa4b251/activemodel/lib/active_model/serialization.rb#L141). It turns out that it's just an alias for `send`!

We can add that into our PORO easily enough:

```
class Conversation
  alias :read_attribute_for_serialization :send
  ...
end
```

Or, we could `include ActiveModel::Serialization` which is where our AR-backed objects got it.

Now when we take a look at `/conversations`, we get:

```
[{"participant":{"id":2,"username":"David","created_at":"2015-02-03T21:04:45.948Z","updated_at":"2015-02-03T21:04:45.948Z"},"messages":[{"sender_id":1,"recipient_id":2,"id":1,"body":"YOLO","created_at":"2015-02-03T21:05:12.908Z","updated_at":"2015-02-03T21:05:12.908Z"},{"sender_id":2,"id":2,"recipient_id":1,"body":"Hello, world!","created_at":"2015-02-03T21:05:51.309Z","updated_at":"2015-02-03T21:05:51.309Z"}]}]
```

Whoops. Not quite right. But the problem is similar to the one we had before in the `MessageSerializer`. Maybe the same approach will work. We'll change the `attributes` to AR relationships.

```
class ConversationSerializer < ActiveModel::Serializer
  has_many :messages, class_name: "Message"
  belongs_to :participant, class_name: "User"
end
```

Almost! Now `/conversations` returns:

```
[{"messages":[{"body":"YOLO"},{"body":"Hello, world!"}],"participant":{"username":"David"}}]
```

We can't see who the sender of each message was! AMS isn't using the `UserSerializer` for the message sender and recipient, because we're not using an AR object.

A little [source code](https://github.com/rails-api/active_model_serializers/blob/master/lib/active_model/serializer.rb#136) spelunking point the way to a fix.

```
class MessageSerializer < ActiveModel::Serializer
  attributes :body, :recipient, :sender

  def sender
    UserSerializer.new(object.sender).attributes
  end

  def recipient
    UserSerializer.new(object.recipient).attributes
  end
end
```

Now `/conversations` gives us what we want:

```
[{"messages":[{"body":"YOLO","recipient":{"username":"David"},"sender":{"username":"Ben"}},{"body":"Hello, world!","recipient":{"username":"Ben"},"sender":{"username":"David"}}],"participant":{"username":"David"}}]
```

And `/messages` still works as well!

## Wrapping Up

The ActiveModel::Serializers gem claims to bring "convention over configuration to your JSON generation". It does a great job of it, but when you need to massage the data, things can get a little bit hairy.

Hopefully some of the tricks we've covered will help you present JSON from your Rails API the way you want. For this, and virtually any other problem caused by _the magic_ getting in the way, I can't suggest digging through the source code enough.

At the end of the day ARS is an excellent choice for getting your JSON API off the ground with a minimum of fuss. Good luck!

P.S. Have a different approach? Prefer `rabl` or `jbuilder`? Did I leave something out? Leave us a comment below!

