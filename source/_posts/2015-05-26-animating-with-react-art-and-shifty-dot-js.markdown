---
layout: post
title: "Animating with React Art and Shifty.js"
date: 2015-05-26 10:38
comments: true
categories: animating react-art react shifty tween
---

Having little experience with D3 animations I'm not sure if it is easy to do animations with D3.
However the great thing about `react` and `react-art` is that in order to animate you follow the same pattern you do for any other rendering, just `setState`.

Animations in CSS3 are different in that a particular element has a defined location and you tell the browser the new location. The inbetween animation state from point a => b over a period of time is automatically handled for you.

In our canvas/svg world we need to `tween` between states. That just means based on a defined time frame (500ms? 1s? 2s?) we need to move an item form `x/y` to a new `x/y`.

[Shifty.js](https://github.com/jeremyckahn/shifty) helps do that in an efficient manner on the web. The reason `shifty.js` works well with React is that it doesn't mutate DOM but just provides you the ability to modify numbers across a space of time. Additionally it provides out of the box easing effects like `elastic`, `bounce`, `linear`, `cubic` and other movements.

This article is less about `react-art` and more about just how to use `shifty.js` since `react-art` is just an extension of `react`, and if you know the fundamental concepts of `react` then you can do just about anything.

Example of a basic tween movement

<!-- more -->


```
var tweenable = new Shifty();
        
        tweenable.tween({
          from:     { x: 50, y: 50},
          to:       { x: 200, y: 200 },
          duration: 1000,
          step: function (state) {
            this.setState({
                x: state.x,
                y: state.y
            })
          }.bind(this)
        });

```


We are saying move from `0,0` to `100,100` over `1000ms (1 second)`.

`Shifty.js` will chunk each step from `0` to `100` over `1000ms` and provide us which each step. 

Yes this looks very much like `jQuery` and it's animate function. They are essentially doing the same thing except `jQuery` modifies the DOM for you and we are just adjusting a number.

Shifty is just one that I happened to pull up, but there are many other tweening libraries that could be used easily.


<p data-height="624" data-theme-id="0" data-slug-hash="bdByVz" data-default-tab="result" data-user="browniefed" class='codepen'>See the Pen <a href='http://codepen.io/browniefed/pen/bdByVz/'>bdByVz</a> by Jason Brown (<a href='http://codepen.io/browniefed'>@browniefed</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

That's the basics, in a later blog post I'll get into some more complex animations. However any tweening library/phsyics engine that is divorced from the DOM will allow you to maniuplate your data and make your `react-art` very versatile.