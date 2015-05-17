---
layout: post
title: "React Native and Socket.io"
date: 2015-05-16 19:35
comments: true
categories: react native websocket socket.io 
---

React Native finally supports Websockets. Which is awesome, however there is one gotcha with socket.io.

1) `npm install socket.io-client`. This will just pull down the socket.io javascript client


2) Everything works great but there is one issue that `window.navigator.userAgent` doesn't exist. Socket.IO checks to deal with some browser incosistencies but we don't care. So all we have to do is create it. However make sure you require `react-native` first!

Just like so

```
var React = require('react-native');

window.navigator.userAgent = "react-native";

var io = require('socket.io-client/socket.io');
```


Now you can do as you please with your new glorious websockets.