---
layout: post
title: "Testing Flux Applications"
date: 2015-09-17 06:29:28 -0600
comments: true
categories:
---

This post originally appeared on [The Quick Left Blog](https://quickleft.com/blog/testing-flux-applications/)

## Introduction

A lot of people in the JavaScript community are pretty excited about Facebook's [React](http://facebook.github.io/react/) library, and associated [Flux](http://facebook.github.io/flux/) architecture. We've been using quite a bit of these tools in our client-side projects at Quick Left. It can be a little hard to wrap your mind around the way the data flows at first, but once you get used to it, you come to appreciate how clean it can be.

As with any development, test-driving features is the way to go in a Flux app. As I've been learning this technology, I've been collecting some of the less obvious patterns that make testing easier. In this post, we'll take a look at some of these strategies, to make it easier for you to build the next big thing.

## Setup

### Project Structure

Since Flux is a more of an idea than a framework, there's no convention as to how to structure your project. I personally like to break it down in a fairly obvious fashion, with the different Flux objects grouped together by folder.

```
app/
├── actions
├── collections
├── dispatchers
├── lib
├── models
├── stores
└── views
```

There are a couple of places to put the tests, but I've been leaning toward a pattern where the tests live right alongside their corresponding files. This makes it easy to find the test for a given module. It also keeps the directory structures from getting out of sync, as they might if you put everything into a separate `test/` folder. Here's an example of what this might look like.

```
app/
├── actions
|   |── user-actions.js
|   |── user-actions-test.js
├── collections
├── dispatchers
├── lib
├── models
├── stores
|   |── users-store.js
|   |── users-store-test.js
└── views
    |── login.js
    |── login-test.js
```

When we find the files to run in our test suite, we can just use [globbing](https://www.npmjs.com/package/glob) to find them all, so it doesn't really present any problems for setting up our tests.

### Testing Dependencies

Facebook recommends using their testing tool, [Jest](https://facebook.github.io/jest/), to test React and Flux components. Although I totally respect Jest, [it doesn't run in the browser](https://github.com/facebook/jest/issues/139), plus I'm pretty used to the toolchain I'm about to describe, so I go about things a slightly different way.

#### Mocha + Chai

When it comes to testing frameworks, I'm a big fan of [Mocha](http://mochajs.org/). It gives us `describe` and all of the other [BDD-style]() assertions we could want when combined with [ChaiJS](http://chaijs.com/).

#### Sinon

Since there are a lot of dependencies in a Flux app, we'll probably be doing a lot of stubbing. I like to use [SinonJS]() for this purpose. It gives us stubs and spies, and its API provides the ability to drill down into how functions were called and with what arguments with a level of granularity that can come in really useful.

#### Karma

When it comes to test runners, there are many viable choices. Lately, I've been leaning toward [Karma]() for most of my needs, because it's easy to get set up, and it can be hooked into a coverage tool with ease.

Here's an example `karma.conf` file for a Flux app in ES6 with [Browserify]() and [Babel]().

```javascript

module.exports = function(config) {
  config.set({
    frameworks: ['mocha', 'browserify'],

    files: [ 'app/**/*-test.js' ],

    preprocessors: {
      'app/**/*.js': [ 'browserify' ]
    },

    browserify: {
      debug: true,
      files: [
        'app/**/*-test.js'
      ],
      transform: [
        ['babelify', { sourceMapRelative: './app' }]
      ]
    },

    browsers: [ 'Chrome' ],

    singleRun: true
  });
};
```

#### Coverage

If you're interesting in setting up a coverage tool to see how well-tested your code base is, check out my post [Measuring Clientside JavaScript Test Coverage With Istanbul](https://quickleft.com/blog/measuring-clientside-javascript-test-coverage-istanbul/).

#### NPM Build Scripts

As far as running tasks, you can rely on Grunt or Gulp, or you can just set a test script up in your `package.json` file. Doing it this way, running tests is as simple as typing `npm test`. Here's what to put into `package.json`:

```json
"scripts": {
  "test": "NODE_ENV=test ./node_modules/karma/bin/karma start"
}
```

With these dependencies set up, you're all ready to start writing tests. Let's take a look at some of the testing specifics.

## Actually Writing Tests

There are four parts to a Flux app: `Actions`, `Stores`, `Views`, and `Dispatchers`.

As mentioned above, Flux is more of a pattern than a framework. Although several people have released experimental frameworks built in its image, there is only one official Facebook package, called `flux`. Ironically, it only contains a `Dispatcher`. You can find the source code [here](https://github.com/facebook/flux). Since this package works well and is tested externally, so we won't be looking at testing `Dispatchers` in this post.

Before we get into looking at the remaining parts of Flux in depth, here are a couple of tips that come in handy in all cases.

### Any Object

#### Setting Up A Sandbox

Sinon sandboxes are a great way to use stubs and spies without having to restore the objects they're touching later. You can clean things up automatically by setting up a new sandbox before each test and tearing it down afterwards.

```javascript
beforeEach(function() {
  this.sinon = sinon.sandbox.create();
});

afterEach(function() {
  this.sinon.restore();
});
```

#### Getting Dependencies

Sometimes it can be a pain to pull in and/or stub a bunch of dependencies for an object you're testing. There's an easy way to grab what you need from within the test: using [Rewire](https://github.com/jhnns/rewire), which exposes a special `__get__` method you can use to access whatever you need from the top level scope of the module. You can then stub out methods and properties on those modules. Here's how to leverage it to your advantage.

```javascript
beforeEach(function() {
  this.sinon = sinon.sandbox.create();
  this.todos = MyAction.__get__('todos');
});

describe('something related to todos', function() {
  it('doesnt have to care about todos', function() {
    this.sinon.stub(this.todos, 'getAll');
    // do something else that calls this.todos.getAll without worrying about the result
  });
});
```

As a note, you don't even need to use Rewire unless you need access to instance variables on your objects. Since Flux uses plain objects, multiple calls to `require` will always return the same object. This means that you can just spy on or stub out a method on one of your `Actions` or `Stores` directly after requiring them.

### Actions

#### Testing Event Dispatching

When you're writing a Flux action, it typically sends some kind of event and payload to the `AppDispatcher` to trigger events registered elsewhere in the application. It's easy to spy on the `AppDispatcher` and test that it's called with the right arguments to ensure that your `Action` is working properly.

```javascript
// my-action.js
import myCollection from '../collections/my-collection';
import AppDispatcher from '../dispatchers/app-dispatcher';

let MyAction = {
  loadModels() {
    myCollection.fetch().then(function() {
      AppDispatcher.dispatch({
        actionType: 'COLLECTION_LOAD'
      });
    });
  }
};

export default MyAction;



// my-action-test.js
import MyAction from './my-action';
import AppDispatcher from '../dispatchers/app-dispatcher';
import Backbone from 'backbone';

it('dispatches an event', function(done) {
  this.spy = this.sinon.spy(AppDispatcher, 'dispatch');
  this.collection = new Backbone.Collection();

  MyAction.loadModels(this.collection);
  setTimeout(function() {
    sinon.assert.calledOnce(this.spy);
    done();
  }, 0)
});
```

#### Testing Promises

We often load data from a remote server in our `Action` objects, so there are typically a lot of promises involved in its internals. When testing these methods, it's often useful to stub out these promises. It's pretty easy to do using native promises in ES6. Note that we use `setTimeout` and `done` to ensure that the promise is fully resolved before testing our assertion and moving on to the next test.

```javascript
// search-action.js
import searchClient from '../lib/search';
import AppDispatcher from '../dispatchers/app-dispatcher';

let SearchAction = {
  search(query) {
    AppDispatcher.dispatch({
      actionType: 'SEARCH_START',
      query
    });
    searchClient.search().then(function(results) {
      AppDispatcher.dispatch({
        actionType: 'SEARCH_SUCCESS',
        payload: results
      });
    });
  }
}

export default SearchAction;



// search-action-test.js
import SearchAction from './search-action';

describe('search', function() {
  beforeEach(function() {
    this.success = new Promise(function(resolve) {
      resolve('results');
    });
    this.failure = new Promise(function(resolve, reject) {
      reject()
    });
    this.searchStub = this.sinon.stub(this.searchClient, 'search')
  });

  it('dispatches a SEARCH_SUCCESS event', function(done) {
    this.appDispatcher = SearchActions.__get__('AppDispatcher');
    this.dispatchStub = this.sinon.stub(this.appDispatcher, 'dispatch');
    this.searchStub.returns(this.success);

    SearchAction.search('my_search');
    setTimeout(function(){
      sinon.assert.calledWith(this.dispatchStub, {
        actionType: 'SEARCH_SUCCESS',
        payload: 'results'
      });
      done()
    }, 0);
  });
});
```

### Stores

In my own Flux projects, I have tried to keep the external API of `stores` as "dumb" as possible. They are meant to be simple repositories for business objects that expose an interface for other objects to subscribe to change events. I typically define methods named `emitChange`, `addChangeListener`, and `removeEventListener` for each store.

Despite their relatively simple API, it is vitally important to test your `stores`. They're usually the place where the business logic lives. Plus, they're responsible for loading data from the server into the client-side app. For these reasons, we want to make sure they work properly. Here are a couple of tricks that can be helpful.

#### Using Internals

Given that `stores` are only supposed to accept data through the callback they register with the `dispatcher`, it can be tricky to send mocked data into them while testing. Facebook has one suggested way of doing it [with Jest](https://facebook.github.io/react/blog/2014/09/24/testing-flux-applications.html#testing-stores), or you can try [this approach](http://bensmithett.com/testing-flux-stores-without-jest/) with Mocha or Jasmine. Alternatively, another nice way to hide the implementation a store uses to fetch its data is to wrap the fetch implementation in an `internals` object and test that instead. Here's what it looks like:

```javascript
import _ from 'lodash';
import {EventEmitter} from 'events;
import Widgets from '../collections/widgets';
import AppDispatcher from '../dispatchers/app-dispatcher';

let WidgetStore = _.extend({}, EventEmitter.prototype, {
  emitChange() {
    this.emit('change');
  },

  addChangeListener(callback) {
    this.on('change', callback);
  },

  removeChangeListener(callback) {
    this.removeListener('change', callback);
  },

  getAll() {
    return this.widgets.toJSON();
  }
});

WidgetStore.internals = {
  init() {
    this.widgets = new Widgets();
    return this.widgets.fetch().then(() => {
      this.emitChange();
    });
  }
};

AppDispatcher.register((action) => {
  switch(action.actionType) {
    case 'INIT_WIDGETS':
      WidgetStore.internals.init();
      break;
    default:
      break;
  }
});

export default WidgetStore;
```

When it comes to testing this `internals` object, we can test that the `internals` methods are behaving as expected. For example:

```javascript
describe('internals', function() {
  beforeEach(function() {
    this.widgets = new Backbone.Collection();
    this.widgets.fetch = function() {
      return new Promise(function(resolve) {
      resolve(['widget1', 'widget2']);
    });
  });

  describe('init', function(done) {
    it('returns a promise', function() {
      WidgetStore.internals.init();
      setTimeout(() => {
        expect(WidgetStore.widgets).to.equal(['widget1', 'widget2']);
        done()
      }, 0);
    });
  });
});
```

#### Using Dependency Injection

If you write your `stores` in an object-oriented way, you can pass a reference to the `dispatcher` directly into them. This makes it easier to test that different dispatching events trigger the correct callbacks to produce the behavior that is desired. Here's an example (thanks to [Jack Hsu](http://jaysoo.ca/2015/03/09/on-flux-stores-and-actions/) for the inspiration for this tip).

```javascript
class WidgetStore extends Store {
  constructor(options) {
    this.widgets = new Backbone.Collection();
    this.dispatcher = options.dispatcher;
    this.dispatcher.register(this.onWidgetAdded);
  }

  onWidgetAdded(action) {
    this.widgets.add(action.payload);
    this.emit('change');
  }
}
```

And now testing the store is simple. We can just inject a dispatcher and use it to trigger the events we want to test.

```
describe('WidgetStore', function() {
  beforeEach(function() {
    this.dispatcher = new Dispatcher();
    this.widgetStore = new WidgetStore({ dispatcher: this.dispatcher });
  });

  describe('WIDGET_ADDED', function() {
    let widget = new Backbone.Model();
    this.dispatcher.dispatch({
      actionType: 'WIDGET_ADDED',
      payload: widget
    });
    expect(this.widgetStore.widgets.toJSON()).to.haveLength(1);
  });
});

```

### View Components

Finally we come to the view layer. If you've played with React, writing these view components should come naturally.

When it comes to testing, there are a lot of things you can safely skip, since testing them would just be verifying that React works as expected. For example, checking that `onClick` handlers fire is pointless, since we know that React will call them. On the other hand, it can be useful that the behavior we want them to cause is actually carried out.

#### Wrap Components With StubRouterContext

It's usually a good idea to wrap your components in a stubbed out context, to make it easier to force them to behave the way you want within your tests. If you don't, it can be hard to get them to render and behave as expected.

To get this to happen, I recommend using the [stub-router-context](https://gist.github.com/cmain/69aee0b5d9cab96589d7) module from the [react-router project](https://github.com/rackt/react-router). It's useful for wrapping the context of all kinds of components aside from the React Router. Although I tend to stick to the name "Stub Router Context", it would perhaps be more accurate to just call it "stub context", since you can use it to stub out any context.

I also like to add a ref to the stub in the component that's returned in `render`, to make it easier to get hold of the component being wrapped by the component returned by `stub-router-context`.

```javascript
render: function() {
  return <Component ref='stub' {...props} />;
}
```

Here's how it looks when you include it in your test. Note how the ref I included makes it easy to grab the child and call `setState` on it.

```javascript
import React from 'react';
import stubRouterContext from '../lib/stub-router-context';
import Search from './search';
let TestUtils = React.addons.TestUtils;

beforeEach(function () {
  let Component = stubRouterContext(Search);
  this.component = TestUtils.renderIntoDocument(<Component/>);
});

it('does the thing when the state is such', function() {
  this.component.refs.stub.setState({
    isLoading: false
  });
});

```
#### Use the React TestUtils

When it comes to rendering your component into a test DOM, checking whether classes are being dynamically added or removed, or whether input values are are changing in response to user interactions, the React TestUtils can't be beat. Get to know [the TestUtils API](https://facebook.github.io/react/docs/test-utils.html), and use it to test your view components. It makes things much less painful.

```javascript
import React from 'react';
import WidgetRepeater from './widget-repeater';
import stubRouterContext from '../lib/stub-router-context';
let TestUtils = React.addons.TestUtils;

describe('WidgetRepeater', function() {
  beforeEach(function() {
    let Component = stubRouterContext(WidgetRepeater);
    // Add 10 midgets to the repeater
    this.component = TestUtils.renderIntoDocument(<Component/>);
  });

  it('shows the widgets', function() {
    let widgets = TestUtils.scryRenderedDOMComponentsWithClass(this.component, 'widget')
    expect(widgets).to.have.length(10);
  });

  it('sets the first widget active', function() {
    let activeWidgets = TestUtils.scryRenderedDOMComponentsWithClass(this.component, 'active')
    expect(activeWidgets).to.have.length(1);
  });

  it('changes widgets to active on click', function() {
    let widgets = TestUtils.scryRenderedDOMComponentsWithClass(this.component, 'widget')
    let secondWidget = widgets[1].findDOMNode();
    TestUtils.Simulate.click(secondWidget);
    let activeWidgets = TestUtils.scryRenderedDOMComponentsWithClass(this.component, 'active')
    expect(activeWidgets).to.have.length(2);
  });
});
```

## Conclusion

Committing to a test-driven development approach in client-side JavaScript applications can sometimes be a hard sell. Aside from problems with testing DOM manipulation and asynchronous code, it can also be hard to test patterns that are new to the team, like Flux. But with the right tools, it can become second nature to test some of these things. Once your team has the confidence that they can effectively test these components, it's a lot easier to approach all feature development with a TDD mindset.

In this post, we explored how to better test Flux applications. We took a look at some of the JS testing tools that can be helpful to get setup in your build process. We talked about some testing tips that are useful across all of the different Flux objects. Then we drilled down into testing tips specific to `Actions`, `Stores`, and `View Components`.

I hope that some of these tips come in useful for your team as you build your cutting-edge web application. Best of luck!
