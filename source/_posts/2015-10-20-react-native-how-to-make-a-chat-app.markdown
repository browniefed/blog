---
layout: post
title: "React Native - How to make a chat app"
date: 2015-10-20 22:07
comments: true
categories: react-native react chat styling dynamic
published: false
---

Chatting online is the epitome of the 90s. Stocking up on AOL discs so you could jump into chat rooms and meet people from across the world. Chat apps have evolved and now we have fancy apps like Slack and Discord to keep in touch.

So what does this have to do with React Native? Well lets build a fancy chat app. I was asked a question about chat bubbles, dynamic text, dynamic widths, and dynamic heights. Of course, the keyword being `dynamic`. Lets dive in and see how to handle this with our dear friend flexbox.

This quesiton was proposed to me, and I'm never one to back down from a challenge, so lets dive in.

## What are we building

## Concept

I will admit, this one took me a while(about an hour) of tweaking to figure out. Apparently some of the good React devs at IBM couldn't figure it out. Not sure what that means but go me?

`ListView => Chats => flexDirection: 'row' => alignSelf magic`. The key to this whole thing is `alignSelf`. Many don't know it's power, but it'll solve all of your problems... in this tutorial.

## Setup

```
var React = require('react-native');
var {
  AppRegistry,
  StyleSheet,
  Text,
  View,
  ListView,
  Animated
} = React;

```

Nothing too surprising. Also `Animated` sneaks in there, because what sort of chat app would I build without a hint of animation.


## The Chat Bubble

## The List of Chats

## Animate the goodness