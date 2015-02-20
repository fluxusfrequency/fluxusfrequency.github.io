---
layout: post
title: "Caching Asynchronous Queries In Backbone"
date: 2014-12-09 06:32:04 -0700
comments: true
categories:
---

This post originally appeared on [Engine Yard](https://blog.engineyard.com/2014/caching-asynchronous-queries-backbone).
It was also published on the [Quick Left Blog](https://quickleft.com/blog/caching-asynchronous-queries-in-backbone-js/).

I was working on a _Backbone_ project with Bob Bonifield recently, when we came across a problem. We were building an administration panel to be used internally by a client, and there were a couple of views that needed to display the same information about users in a slightly different way. To prevent unnecessary AJAX calls, we decided to cache the result of `Backbone.Model#fetch` at two levels of our application.

The result: less network time and a snappier user experience.

Here's how we did it.

## Caching the Controller Call

We decided to use Brent Ertz's [backbone-route-control](https://www.npmjs.org/package/backbone-route-control) package. It makes separation of concerns in the Backbone router easier by splitting the methods associated with each route into controllers.

I'll show how we set up the `UsersController` to handle the first level of caching. In this example, we'll use `backbone-route-control`. If you weren't using it, you could accomplish the same thing in the Backbone router.

First, we set up the app view and initialized a new Router as a property of it, passing in the controllers we wanted to use.

```javascript
// Main

var UsersController = require('controllers/users');

var app = new AppView();

app.router = new Router({
  controllers: {
    users: new UsersController(app)
  }
});
```

Next, we defined the router and set it up to use the UsersController to
handle the appropriate routes.

```javascript
// Router

var BackboneRouteControl = require('backbone-route-control')

var Router = BackboneRouteControl.extend({
  routes: {
    'users/:id': 'users#show',
    'users/:id/dashboard': 'users#dashboard'
  },

  initialize: function(options) {
    this.app = options.app;
  }
});

return Router;
```

## Caching the User at the Controller Level

After we got the router set up, we defined the UsersController and the appropriate route methods. We needed to wait until the user was loaded before we could generate the DOM, because we needed to display some data about the user.

We opted to cache the ID of the last user that was accessed by either the `show` or `dashboard` method, so that we wouldn't repeat the fetch call when we didn't need to. We set the result of the call to `Backbone.Model#fetch` (a promise) to a variable called `userLoadedDeferred`, and passed it down the the views themselves.

In doing so, we took advantage of the fact that, behind the scenes, `fetch` uses [jQuery.ajax](http://api.jquery.com/jquery.ajax/) and returns a [deferred object](http://api.jquery.com/category/deferred-object/). When saving the result of a call to `jQuery.ajax` to variable, the value of the deferred's `.complete` or `.fail` callback will always return the same payload after it has been fetched from the server.

```javascript
// UsersController

var UsersController = function(app) {
  var lastUserID,
      userLoadedDeferred,
      user,
      lastUser;

  return {
    show: function(id) {
      this._checkLastUser();

      var usersView = new UserShowView({
        app: app,
        user: lastUser,
        userLoadedDeferred: userLoadedDeferred
      });

      app.render(usersView);
    },

    dashboard: function(id) {
      this._checkLastUser();

      var usersView = new UserDashboardView({
        app: app,
        user: lastUser,
        userLoadedDeferred: userLoadedDeferred
      });

      app.render(usersView);
    },

    _checkLastUser: function(id) {
      if (lastUserId != id) {
        lastUserId = id;
        lastUser = new User({ id: id });
        userLoadedDeferred = lastUser.fetch();
      }
    }
  }
});
```

## Caching the User at the Model Level

Although our `UsersController` was now caching the result of a fetch for a given user, we soon found that also needed to refetch the user to display their information in a sidebar view as well.

Since the `UsersController` and the `SidebarView` were making two separate calls to the `User` model `fetch` method, we decided to do some more caching in the Backbone Model. We opted to save the results of the `fetch` call for 30 seconds, only making a new server request if the timer had expired.

This allowed us to simply call `fetch` from within the view, without needing to know whether the `User` model was making an AJAX call or just returning the cached user data.

Here's what the code looked like in the model:

```javascript
// User Model

var Backbone = require('backbone');
// Set the timeout length at 30 seconds.
var FETCH_CACHE_TIMEOUT = 30000;

var User = Backbone.Model.extend({
  fetch: function() {

    // set a flag to bust the cache if we have ever set it
    // before, and it's been more than 30 seconds
    var bustCache = !(this.lastFetched && new Date() - this.lastFetched < FETCH_CACHE_TIMEOUT);

    // if we've never cached the call to `fetch`, or if we're busting
    // the cache, make a note of the current time, hit the server, and
    // set the cache to this.lastFetchDeferred.
    if (bustCache) {
      this.lastFetched = new Date();
      this.lastFetchDeferred = Backbone.Model.prototype.fetch.apply(this, arguments);
    }

    // return the promise object that was cached
    return this.lastFetchDeferred;
  }
});

return User;
```

## Busting the Cache

Later on in our development, we came across a situation where we needed to force a new fetch of `User`, right after updating some of their attributes. Because we were caching the result for 30 seconds, the newly updated attributes were not getting pulled from the server on our next fetch call. To overcome this, we neede to bust our cache manually. To make this happen, we changed our overridden `fetch` method to take an option that allowed us to force a refetch.

```javascript
// User Model

  ...

  fetch: function(options) {
    // check if we passed the forceRefetch flag in the options
    var forceRefetch = options && options.forceRefetch;

    ...

    // updated the check to if the flag was passed
    if (!this.lastFetchDeferred || bustCache || forceRefetch) {
    ...
```

## Conclusion

Caching the `User` model in this app reduced our network time by quite a bit. Initially, we were making _two_ server calls per route, because we had to fetch the user to display data in both the main view and the sidebar. After saving the result of the `fetch` in the controller, we were now only calling to the server once per User ID.

With the addition of model-level caching, we were also able to remove the duplicated call between the main views and the sidebar view, by saving the results of the `fetch` call for 30 seconds.

Overall, we reduced four calls per route to one call per 30 seconds. Making these adjustments helped make our application behave a lot more smoothly, and reduced server load in the process.

P.S. Have you implemented anything like this before? What are some of the tricks you use to make Backbone more efficient? Tweet at me @fluxusfrequency.


