---
layout: post
title: "Create a Weather App with React Native"
date: 2016-02-06 12:00
comments: true
categories: react-native, forecast, weather
published: false
---

## Intro

Weather apps are hot, and always heavily and over designed. Turns out you can just look outside... I digress. There is apparently an app called DarkSky that people rant and rave about, and they have an API they use called forecast.io. You can check out the API docs here [https://developer.forecast.io/](https://developer.forecast.io/).

We'll use forecast.io, so sign up for a developer account, and get your API key.

The development of this app was originally intended for a Chicktech workshop. I decided to turn it into a blog post as well just in case anyone wants to follow along.

You can look at all the code here [https://github.com/browniefed/forecast](https://github.com/browniefed/forecast). Each commit is an evolution of the previous code base and each line is commented.

It doesn't implement any data flow like redux because that'd be overkill. This will work for both iOS and Android. We will focus on some slight React basics, but I am assuming you know a little JavaScript and have hopefully done a little React.

Also checkout [https://github.com/catalinmiron/react-native-weather-app](https://github.com/catalinmiron/react-native-weather-app) which uses a weather icon font!


## Setup

Hopefully you have Android/iOS running for React Native and can at least get a basic project working. If you don't then follow the instructions here [http://facebook.github.io/react-native/docs/getting-started.html](http://facebook.github.io/react-native/docs/getting-started.html).

## Start
