---
layout: post
title: "Socket.io on Android + Phonegap"
date: 2014-06-01 08:12
comments: true
categories: socket.io phonegap android webview javascript websockets websocket mobile
---

WebSockets are wonderful, they are one of the cheapest ways to get realtime communications into your web app. They're also a great way to get realtime communications into your mobile application. AJAX is great but wasn't applicable to my needs. Developing PDXLiveBus app required the server to handle all requests to TriMets servers and use AJAX to request the state of particular buses/routes was less realtime than I wanted. I could go more in depth about my usage and experience but the main point of this post is to prevent some table flipping.

I had it all working in browser however it was not working when I did my phonegap serve. It just would not connect, nor even error out ( I don't think I waited long enough). I had been using [https://github.com/mkuklis/phonegap-websocket](https://github.com/mkuklis/phonegap-websocket), which Android WebView (what phonegap utilizes) doesn't support WebSockets until Android KitKat (4.4). I think this is a tremendous oversight, but I digress. I attempted various things to get `phonegap-websocket` to work but I just couldn't.

The real problem wasn't with `phonegap-websocket` it was with my own server code. I had allowed CORS for the `restify` REST API I had setup but had forgotten to allow CORS for Socket.IO. It all came down to this single line of code.

```
io.set( 'origins', '*:*' );
```

Yeah not the most secure but then again I don't think I have a choice since everything is being served from phones and not a particular domain. Also I'm dealing with public transportation vehicle location which doesn't necessarily needing extreme security.

So if you can't get Socket.IO to work on your Android Phonegap app, add `phonegap-websocket` and make sure you have allowed CORS on your server. 