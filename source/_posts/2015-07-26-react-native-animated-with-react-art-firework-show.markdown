---
layout: post
title: "React-Native Animated with React-Art - Firework Show"
date: 2015-07-26 22:12
comments: true
categories: react-native react react-art graphs animated 
published: false
---

#Introduction
This actually could be built with out react-art, but we'll use react-art just for good measure.
What are we building? A firework show. Nothing fancy. Just tap on the screen and a firework will be shot to that point and explode.

#What?

This is what we're building

{% img http://i.imgur.com/Dj60a6e.gif Firework Shooter %}


#Setup

Just like normal, lets set the scene. A blank app with everything imported, etc.

```
var React = require('react-native');
var Dimensions = require('Dimensions');
var ReactNativeART = require('ReactNativeART');

var {
  width,
  height
} = Dimensions.get('window');

var {
  AppRegistry,
  StyleSheet,
  Animated,
  View,
  TouchableWithoutFeedback
} = React;

var {
    Surface,
    Shape,
    Path,
    Group,
    Transform
} = ReactNativeART;

var AnimatedShape = Animated.createAnimatedComponent(Shape);
var AnimatedGroup = Animated.createAnimatedComponent(Group);
```

We bring in the `Animated` API, `ReactNativeART`, some components from `ART`.
The thing that we have done before is creating `AnimatedShape`, and `AnimatedGroup`. What these do is allow us to set props like `fill`, `opacity`, `x`, `y`, that are Animated values. This will cause updates in our native world correctly and efficiently.

Now our styles

```
var styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: 'column'
  }
});
```
Yep that is it. Most of our stuff will take place in the `react-art` world.

Now our basic class

```
var FireworkShooter = React.createClass({
  getInitialState: function() {
    return {
      fireworks: []
    };
  },
  render: function() {
    return (
      <View style={styles.container}>
        <TouchableWithoutFeedback>
          <View>
            <Surface width={width} height={height}>
            </Surface>
          </View>
        </TouchableWithoutFeedback>
      </View>
    );
  }
});
```

So we have a simple setup. A full container view, with a `TouchableWithoutFeedback` so we can get the press coordinates. Then a wrapping `View` since `Touchable` stuff needs a native view below it. Then finally our `react-art` `Surface`. This takes a `width` and `height` prop. In our case we want the full screen, so we get the dimensions of the phone that we extracted up above and set it.


# Shoot a mortar

Lets thing about this. We want a mortar, a small ball, to shoot from the bottom center of the screen to where people tapped.






# Shot multiple mortars that disappear

# Make those mortars explode






#Play with it

Check it out on RNPlay as per usual [https://rnplay.org/apps/ysm12A](https://rnplay.org/apps/ysm12A).


#Final Code