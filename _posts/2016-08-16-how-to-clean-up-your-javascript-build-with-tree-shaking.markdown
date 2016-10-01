---
title: How To Clean Up Your JavaScript Build With Tree Shaking
date: 2016-08-16 02:29:23 Z
categories:
- source
layout: post
comments: true
---

This post originally appeared on [Engine Yard](https://blog.engineyard.com/2016/tree-shaking)

The world of JavaScript development can be maddening and exciting. Every day, new libraries and modules are published, and it can feel overwhelming to try to keep up. On the other hand, all of this change has its benefits. As a community, we're heading more and more toward an ecosystem that's easier to work in and reason about. We keep getting new candy, and it makes our lives better!

One JavaScript improvement that's been getting some attention lately is the idea Tree Shaking. What's that, you ask? Simply put, it's a way to clean up your bundling process by excluding code you're not using. We all hate bloat in our projects. Unused code increases mental overhead and makes it much harder to understand what's going on. More importantly, it increases the size of the payload we're sending to users in front-end projects.

In this post, we'll take a look at tree shaking - what it is, how it works, and how to get started using it. Let's create a cleaner build for your JavaScript project!

## Where Did It Come From?

The idea of tree shaking has begun gaining traction in the JavaScript world thanks to John Doe's [Rollup](https://github.com/rollup/rollup) project. Rollup is a JavaScript module bundler. From early on, it supported EcmaScript 6 (ES6) modules, which resulted in its creating smaller bundles. It also supports some sweet features like cyclical requires and export bindings.

But perhaps the most unique feature of Rollup is the fact that it only requires modules that you've actually imported somewhere. This means that unused code never makes it into the bundle. Note that this is slightly different than Dead Code Elimination. Roman Luitikov does a good job explaining this distinction in his [post about Tree Shaking](https://medium.com/@roman01la/dead-code-elimination-and-tree-shaking-in-javascript-build-systems-fb8512c86edf#.arm0lxyhz).

At this moment, a lot of people in the JS community are coalescing around [Webpack](https://webpack.github.io/) as a build tool. Webpack's maintainers are currently working on a Webpack 2 release. One of its interesting features is that it will support native ES6 modules without first transforming them into a CommonJS format. This is good, because tree shaking won't work with CommonJS modules.

Recently, the estimable Dr. Axel Rauschmayer wrote a popular [post](http://www.2ality.com/2015/12/webpack-tree-shaking.html) about setting up tree-shaking with Webpack 2. I decided to give it a try in my colleague [Marc Garreau](https://twitter.com/omgwtfmarc)'s [Redux Starter Kit](https://github.com/marcgarreau/redux-starter), to see what it would actually take to get tree shaking to work in a full project. You can see the result of my experiments on my [GitHub](https://github.com/fluxusfrequency/redux-starter/tree/tree-shaking).

## How Does It Work?

The steps involved in tree shaking are fairly simple.

You write your ES6 code as normal, importing and exporting modules as needed. When it comes time to create a bundle, Webpack grabs all of your modules and puts them into a single file, but removes the `export` from code that's not being imported anywhere. Next, you run a minification process, resulting in a bundle that excludes any dead code found along the way. If you're curious, check Dr. Rauschmayer's [post](http://www.2ality.com/2015/12/webpack-tree-shaking.html) for more details.

## Setting It Up

Since Webpack 2 is still in beta, you'll need to update your `package.json` to point at the beta version. But before we do that let's also talk about our Babel preset. Typically, I use the `es2015` preset, but this preset relies on the `transform-es2015-modules-commonjs` plugin, which won't work for tree shaking. Dr. Rauschmayer pointed this out in his post. At the time of his writing, the best workaround was to copypasta all of the plugins in that preset except `transform-es2015-modules-commonjs`.

Thankfully, we can now get around this copy-pasta'ing by including the `es2015-native-modules` or `es2015-webpack` preset instead. Both of these presets support native ES6 modules.

Let's install Webpack 2 and the `es2015-native-modules` Babel preset by running `npm install --save babel-preset-es2015-native-modules webpack@2.0.1-beta`.

You should see these packages appear in your `package.json` dependencies section:

```
"dependencies": {
  "babel-preset-es2015-native-modules": "^6.6.0",
  "webpack": "^2.0.1-beta"
  // etc.
}

```

You'll also need to update your `.babelrc` file to use the new preset:

```
{
  presets: ["es2015-native-modules", "react"]
}

```

Remember that we need to run a minification step during our bundle in order to take advantage of dead code elemination. Change the build script to use the `--optimize-minimize` flag on our call to the Webpack executable:

```
// package.json
...
  "scripts": {
    "build": "webpack --optimize-minimize",
    // etc.
  },
...
```

Finally, update your Webpack config to make sure we're only listing one loader for at a time. This is for compatibility with Webpack 2, since it expects a slightly different syntax in the config.

```
// webpack.config.js
...
loaders: [
  {
    test: /\.jsx?$/,
    exclude: /node_modules/,
    loader: 'babel',
    include: __dirname
  },
]
```

## Taking It For A Spin

Now we have everything we need in place to run a build with tree shaking. Let's try it out!

To make sure everything is working, we'll need to export some modules that we're not importing. In my example, I decided to create some useless functions.

```
// place-order.js

export function makeMeASandwich() {
  return 'make sandwich: operation not permitted';
}

export function sudoMakeMeASandwich() {
  return 'one open faced club sandwich coming right up';
}

```

Then in my actual project code, I only imported and used one of them.

```
// index.js

import { sudoMakeMeASandwich } from './place-order.js';
sudoMakeMeASandwich();

```

When we run `npm run build`, we should end up with a minified build that excludes the `makeMeASandwich` code. When you run this command, you'll see an output that accounts for removed modules. In my example, I saw a bunch of warnings from dependencies such as React and Webpack Hot Middleware. I left a few of them in the pasted output below, but the line that's of most interest of us is `Dropping unused function makeMeASandwich`, since that's the code we're checking on.

```
WARNING in bundle.js from UglifyJs
...
Dropping unused function makeMeASandwich [./src/place-order.js:1,16]
...
Dropping unused variable DOCUMENT_FRAGMENT_NODE_TYPE [./~/react/lib/ReactEventListener.js:26,0]
Condition always true [./~/style-loader!./~/css-loader!./~/sass-loader!./src/assets/stylesheets/base.scss:10,0]
Condition always false [(webpack)-hot-middleware/process-update.js:9,0]
Dropping unreachable code [(webpack)-hot-middleware/process-update.js:10,0]
```

Now if we open up the minified bundle and search for the contents of the `makeMeASandwich` function, we won't be able to find it. Indeed, searching for `make sandwich: operation not permitted` yields no results. It works for `one open faced club sandwich coming right up`, though. Success!

## Going Further: Shaking Dependencies

Removing your own unused code is all well and good, but it's probably only going to be a drop in the bucket of your overall bundle. The place where tree shaking really shines is in helping remove usused code from dependencies. If you're pulling in a whole package, but only using a single export from it, why would you send the rest of the package to your users?

Roman Luitikov's [post](https://medium.com/@roman01la/dead-code-elimination-and-tree-shaking-in-javascript-build-systems-fb8512c86edf#.42821257h) did a great job of walking through what it looks like to tree shake a dependency by showing what happens if you pull in Lodash, but only import and use the `first` function. The rest of Lodash gets thrown out, and the resulting build is much smaller!

Since Roman has already covered this, I won't go into the details of what that looks like here, but you can see it in my [example](https://github.com/fluxusfrequency/redux-starter/commit/6ba1ee1455935a8aec054c3a587b0a93d683be5c#diff-1fdf421c05c1140f6d71444ea2b27638R20) if you're curious.

The main point to understand is that when you consider the sizeable amount of unused code that ends up in a typical JS project when you run `npm install`, removing it with tree shaking can yield huge savings on bundle size.

## Conclusion

Being a front-end JavaScript developer comes with a specific set of concerns. Because you're sending a large amount of code execution off onto your user's browser, the size of your payload can get really large. We do our users a huge service by being sensitive to the amount of data that we're sending them.

Tree shaking offers a great way to cut down on your bundle size, and it's easy to get set up in your existing project. I'm excited to see how this practice evolves as more of the community begins to embrace it and Webpack 2 is released. In the meantime, I would recommend reading up on some other [examples of tree shaking](https://github.com/webpack/webpack/tree/master/examples/harmony-unused) and more of the [features coming in Webpack 2](https://gist.github.com/sokra/27b24881210b56bbaff7).

Until next time, happy coding!

P. S. If you decide to give tree shaking a try in your existing project, we'd love to hear how it goes. Leave us a comment!


