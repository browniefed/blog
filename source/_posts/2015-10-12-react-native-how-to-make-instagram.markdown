---
layout: post
title: "React Native - How to make Instagram"
date: 2015-10-12 04:48
comments: true
categories: react-native react instagram gl
published: false
---

Instagram is a fantastic app and a great concept to model after for learning fragment shaders. We won't get too deep into fragment shaders but I'll take a little bit about what they are and point you to some resources.

We'll take advantage of [gl-react-native](https://github.com/ProjectSeptemberInc/gl-react-native) by [Gaëtan Renaudeau](https://twitter.com/greweb), [react-native-view-snapshot](https://github.com/jsierles/react-native-view-snapshot) by [Joshua Sierles](https://twitter.com/jsierles), and [react-native-camera](https://github.com/lwansbrough/react-native-camera) by [Loch Wansbrough](https://twitter.com/lwansbrough)

## Concept

What we're going to do is render various Fragment Shaders over the top of the images that come from the camera roll, or an image that we take in the camera. When we're good and ready, have added/subtract fragment shaders and adjusted levels then we'll simply take a snapshot of the outer wrapping view and save it to an image file. Done! This is a bit contrived (shocker, me writing a contrived blog post?), but it works! This obviously won't support videos, you'll need to do some extra special work for that.

