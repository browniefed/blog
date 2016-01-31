---
layout: post
title: "CSS Modules, Webpack and Testing"
date: 2016-01-31 12:58
comments: true
categories: css modules, css-modules, webpack, testing, react, babel
---

## The Problem

When getting into some basic React testing with CSS Modules I ran into one issue. How the hell do I test them?

You may see the issue that I faced requiring a `.css` file, if you don't let me show you.

```
import React from "react";
import MyCss from "./somecss.css";
import classnames from "classnames";

const MyComponent = ({disabled}) => <div className={classnames(MyCss.container, {[MyCss.disabled]: disabled})} />

export default MyComponent;
```

So there isn't really much to test here, but lets say we wanted to make sure that a `div` had the disabled class when it was given a disabled prop.
We fire up our test, create it, babel compiles, but oh no, `MyCss` is undefined, or an empty object. Well duh, we're using Webpack to process our CSS.

For that matter we could use something to mock the import right? Yeah you could but lets introduce something even more awesome so we don't have to create custom mocks.

## Babel Plugin Webpack Loaders Awesomeness
[Babel Plugin Webpack Loaders](https://github.com/istarkov/babel-plugin-webpack-loaders) allows us to run our tests and process all requires through our typical webpack loaders.

<!-- more -->
So when we're writing our tests to check out if `MyComponent` will have the disabled class we can just `import` the file into our test and know that it'll actually provide the processed CSS Modules!

```
//Test Imports of React, Addons for shallow rendering, teaspoon, maybe enzyme?

import MyCss from "./somecss.css";

//Do some shallow rendering of <MyComponent disabled={true} />

expect(renderedOutput.className).toHaveClass(MyCss.disabled);
```

Yeah that's not actual code, but you get the point. Our [Babel Plugin Webpack Loaders](https://github.com/istarkov/babel-plugin-webpack-loaders) processed all our imports for us and away we go.

What does the setup look like for that?

## Setup

I created a new `runtime.webpack.config.js` file. They even provide you with a basic one, I realized that after the fact. [runtime.webpack.config.js](https://raw.githubusercontent.com/istarkov/babel-plugin-webpack-loaders/master/examples_webpack_configs/run.webpack.config.js)

```
module.exports = {
  output: {
    // YOU NEED TO SET libraryTarget: 'commonjs2'
    libraryTarget: 'commonjs2',
  },
  module: {
    loaders: [
      {
        test: /\.css$/,
        loaders: [
          'style-loader',
          'css-loader?modules&importLoaders=1&localIdentName=[name]__[local]--[hash:base64:5]',
          'postcss-loader',
        ],
      },
    ],
  },
};
```

Created my `.babelrc` file and added in the plugin like so
```
{
  "presets": [
    "es2015",
    "react"
  ],
  "plugins": [
    "transform-object-rest-spread"
  ],
  "env": {
    "test": {
      "plugins": [
        [
          "babel-plugin-webpack-loaders",
          {
            "config": "./runtime.webpack.config.js",
            "verbose": false
          }
        ]
      ]
    }
  }
}
```

You can see we only use this plugin when we set our node environment variable to `test`.

Then for our test running we'll use `mocha` so we'll need to set that up like so

```
scripts: {
  test: "BABEL_DISABLE_CACHE=1 NODE_ENV=test mocha --compilers js:babel-register --recursive"
}
```

Or however you have your testing setup. Moral of the story is set the `NODE_ENV` to `test`, do your `Babel` stuff for testing, and let the magic happen!

Of course this focuses on `css-modules` specifically but all your Webpack plugins will run so test at will!

Also check out [react-component-template](https://github.com/nkbt/react-component-template) by [nkbt](https://github.com/nkbt). A great base for creating a custom react component, using tape for testing, as well as `babel-plugin-webpack-loaders` to process tests.

## Words of warning

Just create a new Webpack config, or a configuration that outputs your app to `libraryTarget` correctly. I opted for a new Webpack config but mine is fairly simple at the moment.

Also if you ever get `ModuleBuildError: Module build failed: TypeError: Cannot read property 'toString' of undefined` that means you have some bad CSS. I should have read a little bit more, was getting issues with `parser` in PostCSS and had left off a closing `}`.

A CSS linter, and or a PostCSS linter probably would have caught this? I don't know, just know that I spent too much time being very confused when it was just human error causing the problem.
