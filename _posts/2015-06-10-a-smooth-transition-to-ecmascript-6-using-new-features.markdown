---
title: 'A Smooth Transition to ECMAScript 6: Using New Features'
date: 2015-06-10 12:28:33 Z
categories:
- source
layout: post
comments: true
---

This post originally appeared on [Engine Yard](https://blog.engineyard.com/2015/smooth-transition-ecmascript-6-new-features)

## Introduction

In [part one of this miniseries](https://blog.engineyard.com/2015/smooth-transition-ecmascript-6-integration), we talked about the timeline for ES6 rollout, feature compatibility in existing environments and transpilers, and how to get ES6 set up in your build process.

Today, we’ll continue the conversation, looking at some of the easiest places to start using ES6 in a typical front-end [Backbone + React](https://blog.engineyard.com/2015/integrating-react-with-backbone) project. Even if that's not your stack, read on! There's something for everyone here.

If you want to try out the examples, you can use a sandboxed ES6 environment at [ES6 Fiddle](http://www.es6fiddle.net/).

## New Features

### Classes, Shorthand Methods, and Shorthand Properties

A lot of client-side JS code is object-oriented. If you're [using Backbone](http://mikefowler.me/2014/06/11/backbone-with-es6/), just about every Model, Collection, View, or Router you ever write will be a subclass of a core library Class. With ES6, extending these objects is a breeze. We can just call `class MySubclass extends MyClass` and we get object inheritance. We get access to a `constructor` method, and we can call `super` from within any method to apply the parent class's method of the same name. This prevents us from having to write things like:

```javascript
Backbone.Collection.prototype.initialize.apply(this, arguments)
```

We also get some handy shorthands for defining methods and properties. Note the pattern I'm using to call `initialize` instead of `initialize: function(args) {}`:

```javascript
class UserView extends Backbone.View {
  initialize(options) {
    super(options);
  }
}
```

We can also define properties using a nice new shorthand. The code below sets an `app` property on the `Injector` that points to the instance of `App` we create on the second line. In other words, it's the same as doing `this.app = app;`.

```javascript
let App = function() {}; // we'll look at 'let' in just a second.
let app = new App();

let Injector = {
  app
};
```

### Let

The new `let` keyword is probably the easiest win that you can possibly get in using ES6. If you do nothing else, just start replacing `var` with `let` everywhere. What's the difference, you ask? Well, `var` is scoped to the closest enclosing _function_, while `let` is scoped to the closest enclosing _block_.

In essence, variables defined with `let` aren't visible outside of `if` blocks and `for` loops, so there's less likelihood for a naming collision. There are other benefits. See [this Stack Overflow answer](http://stackoverflow.com/questions/762011/javascript-let-keyword-vs-var-keyword) for more details.

You can use it pretty much everywhere, but here's a good example of somewhere that it actually makes a difference in preventing a naming collision. The `userName`s inside of the `for` loop don't clash with the current user's `userName` defined just above it.

```javascript
let UserList = React.createClass({
  ...
  render() {
    let userName = this.props.current
    // See the Fat Arrow section below
    let userComponents = this.props.users.map(user => {
      let userName = user.get('userName');
      return <UserComponent displayName={userName} />;
    });
    return (
    <div className="user-list">
      <h1>Welcome back, {userName}!</h1>
      {userComponents}
    </div>
  }
});
```

### Const

As you might guess from the name, `const` defines a read-only (constant) variable. It should be pretty easy to guess where to use this. For example:

```javascript
const DEFAULT_MAP_CENTER = [48.1667, -100.1667];

class MapView extends Backbone.View {
  centerMap() {
    map.panTo(DEFAULT_MAP_CENTER);
  }
}
```

### The Fat Arrow

You've probably already heard about the fat arrow, or used it before if you've written any CoffeeScript. The fat arrow, `=>`, is a new way to define a function. It preserves the value of `this` from the surrounding context, so you don't have to use workarounds like `var self = this;` or `bind`. It comes in really handy when dealing with nested functions. Plus it looks really cool.

```javascript

let Toggle = React.createClass({
  componentDidMount() {
    // iOS
    setTimeout(() => {
      var $el = $('#' + this.props.id + '_label');
      $el.on('touchstart', e => {
        let $checkbox = $el.find('input[type="checkbox"]');
        $checkbox.prop("checked", !$checkbox.prop("checked"));
      });
    }, 0);
  },
```

### Template Strings

Do you ever get sick of doing string concatenation in JavaScript? I sure do! Well, good news! We can finally do string interpolation. This will come in very handy all over the place. I'm especially excited about using it in React `render` calls like this:

```javascript
let ProductList = React.createClass({
  ...
  render() {
    let links = this.props.products.map(product => {
      return (
        <li>
          <a href={`/products/${product.id}`}>{product.get('name')}</a>
        </li>
      );
    });

    return(
      <div className="product-list">
        <ul>
          {links}
        </ul>
      </div>
    );
  }
});
```

### String Sugar

It's always been kind of a pain to check for substrings in JavaScript. `if (myString.indexOf(mySubstring) !== -1)`? Give me a break! ES6 finally gives us some sugar to make this a little easier. We can call `startsWith`, `endsWith`, `includes`, and `repeat`.

```javascript
// Clean up all the AngularJS elements

$('body').forEach(node => {
  let $node = $(node);
  if ($node.attr('class').startsWith('ng')) {
    $node.remove();
  }
});

// Pluralize

function Pluralize(word) {
  return word.endsWith('s') ? word : `${word}s`;
}

// Check for spam

let spamMessages = [];

$.get('/messages', function(messages) {
  spamMessages = messages.filter(message => {
    !(message.toLowerCase().includes('sweepstakes'));
  });
});

// Sing the theme song

let sound = 'na';
sound.repeat(10); // 'nananananananananana'

```

### Argument Defaults

Languages like Ruby and Python have long allowed you to define argument defaults in your method and function signatures. With the addition of this feature to ES6, writing Backbone views requires one less line of boilerplate.

By setting options to an argument default, we don't have to worry about cases where nothing is passed in. No more `options = options || {};` statements!

```javascript
class BaseView extends Backbone.View {
  initialize(options={}) {
    this.options = options;
  }
}
```

### Spread and Rest

Sometimes function calls that take multiple arguments can get really messy to deal with. Like when you're calling them from a `bind` that's being triggered by an event listener.

For example, check out this event listener from a Backbone view in a recent project I was working on. Because of the method signature of `_resizeProductBox`, I have to pass all those null arguments into `bind` and it gets kind of ugly.

```javascript
class ProductView extends BaseView {
  initialize() {
    this.listenTo(options.breakpointEvents, 'all',
      _.bind(this._resizeProductBox, this, null, null, true));
  }

  _resizeProductBox(height, width, shouldRefresh) {
    ...
  }
}
```

In ES6, we can clean this up a bit with _spread_. We'll just prepend an array of default arguments with a `...`  to send them through as arguments to the method call.

Here's how you'd do it using _spread_:

```javascript
const BREAKPOINT_RESIZE_ARGUMENTS = [null, null, true];

class ProductView extends BaseView {
  initialize() {
    this.listenTo(options.breakpointEvents, 'all',
      _.bind(this._resizeProductBox, this, ...BREAKPONT_RESIZE_ARGUMENTS));
  }

  _resizeProductBox(height, width, shouldRefresh) {
    ...
  }
}
```

On the other side of the coin is _rest_, which lets us accept any number of arguments in a method signature instead of at invocation time, as you can do with the _splat_ in Ruby. For example:

```javascript
function cleanupViews(...views) {
  views.forEach(function(view) {
    view.remove();
  });
}
```

### Array Destructuring

Sometimes I find myself having to access all of the elements of an array-like object with square brackets. It's kind of a bummer. Luckily, ES6 lets me use array destructuring instead. It makes it easy to do things like splitting latitude and longitude from an array into two variables, as I do here. (Note that I also could have used _spread_.)

```javascript
let markers = [];

listings.forEach(function (listing, index) {
  let lat, lng = listing.latlng; // looks like: [39.719121, -105.191969]
  let listingMarker = new google.maps.Marker({
    position: new google.maps.LatLng(lat, lng)
  });
  markers.push(listingMarker);
});
```

### Promises

It seems like every project I work on these days uses promises. Native promises have landed in ES6, so we can all rely on the same API from here on out. Both promise instances and static methods like `Promise.all` are provided. Here's an example of a `userService` from an Angular app that returns a promise from `$http` if the user is online, and a native promise otherwise.

```javascript
angular.module('myApp').factory('userService', function($http, offlineStorage) {
  return {
    updateSettings: function(user) {
      var promise;

      if (offlineStorage.isOffline()) {
        promise = new Promise(function (resolve, reject) {
          resolve(user.toJSON());
        });
      } else {
        promise = $http.put('/api/users/' + user.id, user.settings)
        .then(function(result) {
          return result.data;
        });
      }

      return promise.then(data => {
        return new User(data);
      });
    }
  };
});
```

## A Note On Modules

You may have noticed that I didn't cover modules, importing, or exporting in this miniseries. Although modules are one of the higher profile features in ES6, and they're easy to get started with, they still have a lot of edge cases that need to be worked out as ES6 is rolled out.

Specifically, ES6 modules have a `default` export and named exports. CommonJS and AMD only support a single export, and traditionally handle named exports by exporting an object with the named exports as properties. The different ES6 module libraries have different ways of reconciling the differences, so you have to use only default exports or named exports if CommonJS might use the module. How these differences will be reconciled remains to be seen.

That said, the easiest way to start requiring modules is to switch from:

```javascript
var _ = require(‘lodash’);
````

To the new syntax:

```javascript
import _ from ‘lodash’;
```

Similarly, you can import relative files using:

```javascript
import Router from ‘../router’;
```

When exporting, you can switch from:

```javascript
module.exports = App;
```

To this:

```javascript
export default App;
```

There’s a bit more to using modules (such as named exports), which we won’t cover today. If you want to learn more, take a look at [this overview](https://babeljs.io/docs/learn-es6/#modules).

## Conclusion

In this miniseries, we took a look at some real-world examples of how you would use ES6 in a client-side JavaScript app. We took a quick look at setting up an ES6 transpile step in your build process, and examined many of the easy-to-use features you can start using right away.

Before you know it, ES6 will be the standard language in use across the web and on servers and personal computers everywhere. Hopefully, this walkthrough will help you get started with a minimum of fuss.

If you're feeling excited about ES6, and you want to learn more, I would suggest reading through [this overview](https://babeljs.io/docs/learn-es6/). Or if you're feeling really enthusiastic, try [this book](https://github.com/getify/You-Dont-Know-JS/tree/master/es6%20%26%20beyond).

Until next time, happy coding!

