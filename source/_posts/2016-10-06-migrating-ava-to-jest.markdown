---
layout: post
title: "Migrating Ava to Jest"
date: 2016-10-06 18:48
comments: true
categories: jest testing ava react
---

First off. Star the repo here [https://github.com/facebook/jest](https://github.com/facebook/jest) so you can show some love for the hardwork that has been put into it.

### Props

I was an early adopter of Jest back in the early days ( ~v0.4). It was okay but difficult to setup and slow with sometimes weird results and behaviors.

Tireless effort has been done by [Christopher Pojer](https://twitter.com/cpojer) to get to the latest Jest (v16.01) so props to him and others working with him.

Check out the fantastic documentation here [https://facebook.github.io/jest/docs/getting-started.html](https://facebook.github.io/jest/docs/getting-started.html#content)

### Migrating in seconds

There is almost nothing to write about on this topic because it was almost too easy.
Mostly because of [jest-codemods](https://github.com/skovhus/jest-codemods). It is a set of codemods crafted to transition your Tape or Ava project to Jest.

It is worth it. Jest has come along way when it comes to speed, functionality, configuration options, and ease of configuration (or lack there of to get started).

There were minor configurations I had to setup, but most were copying and pasting from the documentation.

Lets dive into the process real quick.

### First and last step to transition tests

First install `jest-codemods`

`npm install -g jest-codemods`

Then run `jest-codemods` in a desired directory.
It will prompt you with `Ava`, `Tape`, or a `both` option.
Then you provide a glob. I was running in my root so I provided the glob `**/__tests__/**/*.js` to convert all of my tests.

That was it.

<!-- more -->
### Done

For basic implementations that is all you need!

Add `"test": "jest"` to your `package.json` `scripts` section and you're good to go!

### Caveats

There are a few caveats to note. 
In Ava you can pass in custom messages to the assertions. This is missing in the default `expect` provided by Jest but is by no means a deal breaker.

`t.plan` and `t.skip` aren't supported by the codemod.

`t.skip` and `t.plan` go hand-in-hand. I find `t.skip` to be somewhat unnecessary but `t.plan` is great for async tests.

This isn't a deal breaker by any means. And Christopher is happy to accept a PR for it too!.
 
### What is awesome

 Our setup uses Webpack `root` property to use shared code across 2 directories in a monorepo. Adding that in other testing libraries required webpack and other NPM installed libraries.
 In Jest it is as simple as adding this to your config.


```
 "modulePaths": [
    "/shared"
  ],
```

Are you using CSS Modules and want to actually test out certain classes get applied?
Well Jest has a nice out of the box solution. It does require an additional NPM install of `identity-obj-proxy` but it's worth when it's an easy config option.

```
"moduleNameMapper": {
    "^.+\\.(css|less)$": "identity-obj-proxy"
}
```

That's all you need to do to get CSS modules working.

For more awesome-examples and full configuration options checkout the documentation [https://facebook.github.io/jest/docs/getting-started.html](https://facebook.github.io/jest/docs/getting-started.html).
Many of the normal uses cases have been though of and are documented with copy and paste configuration options!

### Check it out

I'm barely scratching the surface of what is great about Jest. 
There is hopefully more coming but go use Jest, star the repo on git [https://github.com/facebook/jest](https://github.com/facebook/jest) and never look back.