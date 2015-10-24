---
layout: post
title: "React Native - Recreating the Apple TV Icons"
date: 2015-10-23 20:53
comments: true
categories: react-native react tvos apple tv icons
---

I had attempted to recreate this in the past but could never come up with anything elegant. I saw this post not too long ago [Recreating the Apple TV icons in JavaScript and CSS](https://medium.com/@nashvail/recreating-the-apple-tv-icons-in-javascript-and-css-eec306d41617) by [Nash Vail](https://twitter.com/NashVail).

He then went on to create a [jQuery plugin](https://github.com/nashvail/ATVIcons) to accomplish the effect. After reading the source and viewing the demos, it turns out re-making this in React Native is trivial.

So read the article, check out the [live demo](http://nashvail.me/ATVIcons/) here and then we'll continue on.


## What are we building

{% img http://i.imgur.com/3TLQtmE.gif No 3D glasses required %}

<!-- more -->

## Setup

```
var React = require('react-native');
var {
  AppRegistry,
  StyleSheet,
  Text,
  View,
  PanResponder,
  Image,
  Animated
} = React;

var width = 280;
var height = 150;

```

Yes, Animated again. Can you believe I almost wrote this tutorial with out it? I had a `setState` implementation but I didn't take the lazy, non-performant way out, I built it with performance in mind! No thanks necessary, it would have weighed on my conscience had I released an animated tutorial using `setState`.

The card we're animating is `280` by `150`. It'll play into our calculations. This could be made dynamic though.

## Defaults

```
  getInitialState: function() {
    return {
      maxRotation: 12,
      maxTranslation: 6,
      perspective: 800
    };
  },
```
In our `getInitialState` we'll set up some defaults. We'll set our `maxRotation` to 12, this means the card can only rotate a maximum of 12 degrees. `maxTranslation` is the same thing, it can only shift the card a maximum of 6. 

You can read more about `perspective` here [https://developer.mozilla.org/en-US/docs/Web/CSS/perspective](https://developer.mozilla.org/en-US/docs/Web/CSS/perspective).

```
The perspective CSS property determines the distance between the z=0 plane and the user in order to give to the 3D-positioned element some perspective
```

## Borrowed Function

```
function calculatePercentage(offset, dimension) {
  return ((-2 / dimension) * offset) + 1;
}
```

This is the magic formula. Based upon what we pass in here it will spit out a value between `-1` and `1`. We use this to multiply by our `maxRotation` or `maxTranslation` to get the degree to apply. We won't do the multiplication though, we'll let `Animated` take care of that.


## PanResponder and Calculations

```
  componentWillMount: function() {
    this._animations = {
      xRotationPercentage: new Animated.Value(0),
      yRotationPercentage: new Animated.Value(0),
      xTranslationPercentage: new Animated.Value(0),
      yTranslationPercentage: new Animated.Value(0)
    }

    this._panResponder = PanResponder.create({
      onStartShouldSetPanResponder: (evt, gestureState) => true,
      onStartShouldSetPanResponderCapture: (evt, gestureState) => true,
      onMoveShouldSetPanResponder: (evt, gestureState) => true,
      onMoveShouldSetPanResponderCapture: (evt, gestureState) => true,
      onPanResponderMove: (e, gestureState) => {
        e.persist();
        var {
          locationX: x,
          locationY: y
        } = e.nativeEvent;

        this._animations.xRotationPercentage.setValue(calculatePercentage(y, height));
        this._animations.yRotationPercentage.setValue(calculatePercentage(x, width) * -1);
        this._animations.xTranslationPercentage.setValue(calculatePercentage(x, width));
        this._animations.yTranslationPercentage.setValue(calculatePercentage(y, height));
      }
    })
  },
```
Alright there is sort of a lot here but not really. We setup an object to hold our animations called `this._animations`. We then setup our `PanResponder` defaults, and of course the one we care about is `onPanResponderMove`.

Here when it moves we get the `locationX` and `locationY` which is the `x/y` values relative to the `component` we attach it too.

Finally we run our calculations and call `setValue` on each `Animated.Value`. This is basically what `Animated.event` is doing under the hood for us, but instead we are performing calculations and calling `setValue` ourselves.


## Render-me-timbers

```
  render: function() {
    return (
      <View style={styles.container}>
        <View style={{width: width, height: height}} {...this._panResponder.panHandlers}>
          <Card  
              style={{backgroundColor: 'red', width: width, height: height}}
              stackingFactor={1}
              {...this.state}
              {...this._animations}
          >
            <Card 
              style={{backgroundColor: 'black', position: 'absolute', top: 30, left: 90, width: 100, height: 100}}
              stackingFactor={1.4}
              {...this.state}
              {...this._animations}
            >
              <Card
                style={{backgroundColor: 'yellow', position: 'absolute', top: 20, left: 20, width: 63, height: 63}} 
                stackingFactor={1.8}
                {...this.state}
                {...this._animations}
              />
            </Card>
          </Card>
        </View>
      </View>
    );
  }
```
We'll get to the card component in a second. We attach our `PanResponder` we created to the outer wrap object. Then we nest each card. This is crucial otherwise our layer will 3D transform behind the red layer. 
So this is forcing the `Card` to sit in front of the previous card no matter what. I don't know if this is true, but it was happening to me and this is how I fixed it.

We use a prop called `stackingFactor`. All this does is slightly amplify that cards movements more causing a slight offset and the 3D effect.

```
var Card = React.createClass({
  componentWillMount: function() {

  var translateMax = (this.props.maxTranslation * this.props.stackingFactor);
  var rotateMax = this.props.maxRotation;

    this._xRotation = this.props.xRotationPercentage.interpolate({
      inputRange: [-1, 1],
      outputRange: [ (rotateMax * -1) + 'deg', rotateMax + 'deg'],
      extrapolate: 'clamp'
    });

    this._yRotation = this.props.yRotationPercentage.interpolate({
      inputRange: [-1, 1],
      outputRange: [ (rotateMax * -1) + 'deg', rotateMax + 'deg'],
      extrapolate: 'clamp'
    });

    this._translateX = this.props.xTranslationPercentage.interpolate({
      inputRange: [-1, 1],
      outputRange: [(translateMax * -1), translateMax],
      extrapolate: 'clamp'
    })

    this._translateY = this.props.yTranslationPercentage.interpolate({
      inputRange: [-1, 1],
      outputRange: [(translateMax * -1), translateMax],
      extrapolate: 'clamp'
    })

  },

  getTransform:function() {
    return [
      {perspective: this.props.perspective},
      {rotateX: this._xRotation},
      {rotateY: this._yRotation},
      {translateX: this._translateX},
      {translateY: this._translateY},
    ]
  },
  render: function() {
    return (
      <Animated.View 
        {...this.props} 
        style={[this.props.style, {transform: this.getTransform()}]}
      >
        {this.props.children}
      </Animated.View>
    )
  }
})

```

That's a lot of `interpolate`! Well not really they're mostly all doing the same thing here. Remember when I said our calculations up above could return a percentage between `-1` and `1`.

Well all we have to do is specify that as our `inputRange` and then our `outputRange` is just whatever our small `maxRotation` or `maxTranslation`. `Animated` will take care of all the multiplication for us! Thanks Animated!

The `extrapolate: 'clamp'` is EXTREMELY important. Without it the values will go past their maximums. This can be accomplished since `locationX` and `locationY` could go beyond the `width` and `height` of the container. Long story short, `extrapolate: 'clamp'` your interpolations!

You can see how the `translateMax` is affected when our `stackingFactor` is larger.

```
  var translateMax = (this.props.maxTranslation * this.props.stackingFactor);
```

Our `getTransform` just maps the appropriate animated value to it's transform. Also so our `Card` can be nested we have to specify `{this.props.children}`. 


## Look Ma' no setState

That's right. We've used all `Animated` here. No diffs are happening to cause re-render, so our animations should be quite performant.


## Done

This is currently only supported on iOS but appropriate support for Android is being added here [https://github.com/facebook/react-native/pull/3522](https://github.com/facebook/react-native/pull/3522)

### As always live code [https://rnplay.org/apps/qLNwNw](https://rnplay.org/apps/qLNwNw)

{% img http://i.imgur.com/3TLQtmE.gif Cheaper than going to the movies %}

## Full Code

```
var React = require('react-native');
var {
  AppRegistry,
  StyleSheet,
  Text,
  View,
  PanResponder,
  Image,
  Animated
} = React;


var width = 280;
var height = 150;

function calculatePercentage(offset, dimension) {
  return ((-2 / dimension) * offset) + 1;
}

var AppleTV = React.createClass({
  getInitialState: function() {
    return {
      maxRotation: 12,
      maxTranslation: 6,
      perspective: 800
    };
  },
  componentWillMount: function() {
    this._animations = {
      xRotationPercentage: new Animated.Value(0),
      yRotationPercentage: new Animated.Value(0),
      xTranslationPercentage: new Animated.Value(0),
      yTranslationPercentage: new Animated.Value(0)
    }

    this._panResponder = PanResponder.create({
      onStartShouldSetPanResponder: (evt, gestureState) => true,
      onStartShouldSetPanResponderCapture: (evt, gestureState) => true,
      onMoveShouldSetPanResponder: (evt, gestureState) => true,
      onMoveShouldSetPanResponderCapture: (evt, gestureState) => true,
      onPanResponderMove: (e, gestureState) => {
        e.persist();
        var {
          locationX: x,
          locationY: y
        } = e.nativeEvent;

        this._animations.xRotationPercentage.setValue(calculatePercentage(y, height));
        this._animations.yRotationPercentage.setValue(calculatePercentage(x, width) * -1);
        this._animations.xTranslationPercentage.setValue(calculatePercentage(x, width));
        this._animations.yTranslationPercentage.setValue(calculatePercentage(y, height));
      }
    })
  },
  render: function() {
    return (
      <View style={styles.container}>
        <View style={{width: width, height: height}} {...this._panResponder.panHandlers}>
          <Card  
              style={{backgroundColor: 'red', width: width, height: height}}
              stackingFactor={1}
              {...this.state}
              {...this._animations}
          >
            <Card 
              style={{backgroundColor: 'black', position: 'absolute', top: 30, left: 90, width: 100, height: 100}}
              stackingFactor={1.4}
              {...this.state}
              {...this._animations}
            >
              <Card
                style={{backgroundColor: 'yellow', position: 'absolute', top: 20, left: 20, width: 63, height: 63}} 
                stackingFactor={1.8}
                {...this.state}
                {...this._animations}
              />
            </Card>
          </Card>
        </View>
      </View>
    );
  }
});


var Card = React.createClass({
  componentWillMount: function() {

  var translateMax = (this.props.maxTranslation * this.props.stackingFactor);
  var rotateMax = this.props.maxRotation;

    this._xRotation = this.props.xRotationPercentage.interpolate({
      inputRange: [-1, 1],
      outputRange: [ (rotateMax * -1) + 'deg', rotateMax + 'deg'],
      extrapolate: 'clamp'
    });

    this._yRotation = this.props.yRotationPercentage.interpolate({
      inputRange: [-1, 1],
      outputRange: [ (rotateMax * -1) + 'deg', rotateMax + 'deg'],
      extrapolate: 'clamp'
    });

    this._translateX = this.props.xTranslationPercentage.interpolate({
      inputRange: [-1, 1],
      outputRange: [(translateMax * -1), translateMax],
      extrapolate: 'clamp'
    })

    this._translateY = this.props.yTranslationPercentage.interpolate({
      inputRange: [-1, 1],
      outputRange: [(translateMax * -1), translateMax],
      extrapolate: 'clamp'
    })

  },

  getTransform:function() {
    return [
      {perspective: this.props.perspective},
      {rotateX: this._xRotation},
      {rotateY: this._yRotation},
      {translateX: this._translateX},
      {translateY: this._translateY},
    ]
  },
  render: function() {
    return (
      <Animated.View 
        {...this.props} 
        style={[this.props.style, {transform: this.getTransform()}]}
      >
        {this.props.children}
      </Animated.View>
    )
  }
})

var styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  }
});

AppRegistry.registerComponent('rn_dragtoshow', () => AppleTV);
```
