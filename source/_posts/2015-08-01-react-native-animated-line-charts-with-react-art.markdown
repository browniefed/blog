---
layout: post
title: "React-Native Animated Line Charts with React-Art"
date: 2015-08-01 10:12
comments: true
categories: react-native react react-art animated line chart graph animation
published: false
---

# Introduction

Once again this is another post inspired by a github issue. Less of an issue, more of a question but check it out here [https://github.com/facebook/react-native/issues/1919](https://github.com/facebook/react-native/issues/1919). 

The question asked if you could get the total length of a path. The answer, yes. Using `MetricsPath` from the ART library.


#Strategy

The `strokeDash` property of a particular shape takes a single value, or alternatively an array. When given an array it will expose only the segments of that path. Or from MDN "A list of numbers that specifies distances to alternately draw a line and a gap". In canvas there is a function called `setLineDash` and in SVG it is a `strokedash-array` property. However they operate the same way.

Check out the MDN docs on it [https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/setLineDash](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/setLineDash).

So if we can create a line chart. Get the full path, we can instruct React-Art to expose a segment of the line path every few milliseconds. Each animation step we expose a little more, and a little more, thus making the line look animated.

<!-- more -->


# Setup

We'll use something to generate the line chart for us. That is the fantastic library `paths-js`. It will genereate all the SVG paths and doesn't enforce any special rendering.

So lets do an `npm install paths-js`.

Also we'll need to do an `npm install art` to access the `MetricsPath`