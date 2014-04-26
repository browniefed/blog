---
layout: post
title: "Ractive.JS - Next Generation DOM Manipulation"
date: 2014-02-17 18:34
comments: true
categories: 
---

Next generation DOM manipulation is right mainly because you don't have to do any. Ractive uses declarative binding(like Knockout and Angular) but also uses reactive programming methods (like React). The different is that it does do DOM diffing like React or dirty checking like Angular but utilizes dependancy tracking. Meaning it tracks the changes you make and will update the DOM accordingly. Now there are inherint issues in that ou can take performance hits if you have a large data set and call "set" within a loop. This will hit the DOM very hard since the nature of dependancy tracking says that "when X changes update Y". 