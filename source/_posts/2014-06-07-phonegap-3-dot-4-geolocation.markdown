---
layout: post
title: "Phonegap 3.4 GeoLocation"
date: 2014-06-07 08:28
comments: true
categories: 
---

It doesn't appear to be just me but PhoneGap can be incredibly difficult to use. Simply because any information pertaining to the most current version of phonegap is so fragmented. A google search will reveal virtually nothing about the error messages you're receiving and how to remedy them for current versions.

For example, I was attempting to use the GeoLocation API. Could care less about the extreme accuracy. After attempting to use it I was getting an Error Code 2 , failed to start geolocation service. Well maybe that was my fault for enabling `enableHighAccuracy` but wait you have to enableHighAccuracy for older versions of android otherwise it won't work. Or was that only for older versions of the geolocation plugin which then got deprecated in some versions of android in favor of the native browser geolocation API because it provides better and faster GPS locations. In the end the fault was all mine, I had simply forgotten to include the plugin in my `config.xml` 

```
    <gap:plugin name="org.apache.cordova.geolocation" version="0.3.8" />
```

Version `0.3.8` was the current version at the time of this article. Actually released on June 5th, 2014 just before the article written on June 6th, 2014.

I've sort of determined that the only real reliable way to determine anything is just to look at the release notes on github [https://github.com/apache/cordova-plugin-geolocation/blob/master/RELEASENOTES.md](https://github.com/apache/cordova-plugin-geolocation/blob/master/RELEASENOTES.md) for the best most up to date information.

To end I should say I really love PhoneGap and all of the time and effort people are putting into developing it. The documentation and information about current versions is fragmented, and rather than ranting I should be helping out and contributing to documentation and solving the problems. That should always be the goal if you find a problem in the open source world. Don't rant, help out, and that is what I will do.