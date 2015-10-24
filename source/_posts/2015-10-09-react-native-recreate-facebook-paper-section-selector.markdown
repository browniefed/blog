---
layout: post
title: "React Native - Recreate Facebook Paper Section Selector"
date: 2015-10-09 20:23
comments: true
categories: react-native react facebook paper animated
published: false
---

Someone said React Native probably couldn't and or wasn't well suited for building something like Facebook Paper. Well you know what happens when someone says something like that... someone has to go out and do it. Figured it might as well be me.

## Interaction Breakdown

Alright lets breakdown the interactions. The view comes in 2 parts. The top part is selected topics, and the bottom part is topics that aren't selected.

The top section can scroll left/right and each card is slightly wobbling. The top cards can then be dragged downwards towards the "unselected" topics. This causes it to stop wobbling and scale up. If it's moved far enough downward it can be dropped, and re-added to the "unselected" topics seamlessly.

The bottom "unselected" section can also be slide left/right, however each of the cards slowly rotates and disappears so you can see only 3 cards at a time maximum. Which ever card is closest to the center will show a name, and a description above it.

If the card is dragged upwards, a "Drag here" section will open up in the "selected" topics section. This will appear at the closest direct position above the card. Since we can only see a maximum of 3 cards that means it could appear in 3 different locations, left, center, or right. When the "Drag here" appears X number of cards slide left.

Yikes. We have to build that.


## Create the interface

Aka. Lets add a blurry background photo :).

## Start simple - Make a card


## Make it Wobble

We'll make a `Wobble` component. I had this integated into the `Card` component above but this will basically be similar to `TouchableOpacity`, it's an animated view wrapper. It just makes things a hell of a lot easier to deal with.
