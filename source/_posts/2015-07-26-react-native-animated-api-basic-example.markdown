---
layout: post
title: "React-native Animated API Basic Example"
date: 2015-07-26 11:51
comments: true
categories: react-native animated api react
---

#Introduction

Animations are finally solved in React? That's a bold claim, but lets explore the new Animated API in `react-native`. This won't apply to React for the web, however there is also [react-motion](https://github.com/chenglou/react-motion) also released at `react-europe`. 


{% img http://i.imgur.com/JlX4nV0.gif Final Animation Result %}

#Resources

* [Cheng Lou - The State of Animation in React at react-europe 2015](https://www.youtube.com/watch?v=1tavDv5hXpo)
* [react-motion - Github](https://github.com/chenglou/react-motion)
* [React Native Animation API](https://facebook.github.io/react-native/docs/animations.html#content)
* [Spencer Ahrens - React Native: Building Fluid User Experiences at react-europe 2015](https://www.youtube.com/watch?v=xDlfrcM6YBk)

#How It Works

The Animated API does not depend on state. It does, however it is not accomplished by using `setState`. The Animated API exports a few components `Animated.View`, `Animated.Text`, and `Animated.Image`. The Animated API will adjust the components in the native Objective-C world. This will bypass the diff and reconciliation in the JS world so you get fluent, and performance animatons. Ultimately all you need to know is that it will interpolate numbers and update the native view components.

#Cool Examples

* [https://github.com/brentvatne/react-native-animated-demo-tinder](https://github.com/brentvatne/react-native-animated-demo-tinder)
* [UIExplorer Animated example](https://github.com/facebook/react-native/tree/master/Examples/UIExplorer/AnimationExample)

# Simple Move Around the Screen Example

We are just going to move a square around the edges of the screen.

```
<--<           
|  |          
V--^         
```

# Setup

Dependencies

```
var React = require('react-native');
var Dimensions = require('Dimensions');

var {
  width,
  height
} = Dimensions.get('window');

var {
  AppRegistry,
  StyleSheet,
  View,
  Animated
} = React;

var SQUARE_DIMENSIONS = 30;

```

Styles

```
var styles = StyleSheet.create({
  container: {
    flex: 1
  },
  square: {
    width: SQUARE_DIMENSIONS,
    height: SQUARE_DIMENSIONS,
    backgroundColor: 'blue'
  } 
});
```

Basic Code

```

var AnimatedSquare = React.createClass({
  getInitialState: function() {
    return {
        pan: new Animated.ValueXY()
    };
  },
  getStyle: function() {
    return [
              styles.square, 
              {
                transform: this.state.pan.getTranslateTransform()
              }
            ];
  },
  render: function() {
    return (
      <View style={styles.container}>
        <Animated.View style={this.getStyle()} />
      </View>
    );
  }
});
```

Few things to call out here.

Notice our state that is created is an instantiation of `Animated.ValueXY`. This will save us some code, and let the `Animated` API take care of interpolating both our X, and Y values.

Our `getStyle` will return an array, our base `square` class and a `transform`. Once again we'll use a the `getTranslateTransform` helper from the `Animated` API to return the appropriate structure for the transform style. 

It returns `[{ translateX: xValue}, {translateY: yValue}]`, where `xValue` and `yValue` are the interpolated values from the `Animated.ValueXY` we set on our `pan` state variable.

Finally we will use the `Animated.View` which is a convenience element to say "Hey React this is going to be an animated thing".


# Move It

We're now going to move it from the top corner. `x = 0, y = 0` to the bottom left corner ` x = 0, y = (phoneHeight - square Height)`.

```
var SPRING_CONFIG = {tension: 2, friction: 3}; //Soft spring

//...
  componentDidMount: function() {
    Animated.spring(this.state.pan, {
          ...SPRING_CONFIG,
          toValue: {x: 0, y: height - SQUARE_DIMENSIONS}                        // return to start
    }).start();
  },
```

So on mount we'll start a spring. This will animate the `this.state.pan` to the bottom left corner like we want it to.

We'll also setup the `SPRING_CONFIG` with a soft spring, not much tension or friction. So it'll get to the corner and just bounce a little bit and stay there.


# Move It, Move It, and Move It again

We can queue up sequences of animations. These will happen one after the other.
The `sequence` call is one of the means of composing animations. There is also `parallel` which allows for declaration of animations to happen at the same time.

```
  componentDidMount: function() {
    Animated.sequence([
      Animated.spring(this.state.pan, {
            ...SPRING_CONFIG,
            toValue: {x: 0, y: height - SQUARE_DIMENSIONS} //animate to bottom left
      }),
      Animated.spring(this.state.pan, {
          ...SPRING_CONFIG,
          toValue: {x: width - SQUARE_DIMENSIONS, y: height - SQUARE_DIMENSIONS} // animated to bottom right
      }),
      Animated.spring(this.state.pan, {
            ...SPRING_CONFIG,
            toValue: {x: width - SQUARE_DIMENSIONS, y: 0} //animate to top right
      }),
      Animated.spring(this.state.pan, {
          ...SPRING_CONFIG,
          toValue: {x: 0, y: 0} // return to start
      })
    ]).start();
  }
```

We define 4 spring configrations like discussed before. The comments in the code explain each movement.


# Move and Repeat

The call to `start` takes a callback. This callback will be invoked once the animation is completed. In our case the animation is complete once we get back to the start. We can then restart the animation.

```
  componentDidMount: function() {
    this.startAndRepeat();
  },
  startAndRepeat: function() {
    this.triggerAnimation(this.startAndRepeat);
  },
  triggerAnimation: function(cb) {
    Animated.sequence([
      Animated.spring(this.state.pan, {
            ...SPRING_CONFIG,
            toValue: {x: 0, y: height - SQUARE_DIMENSIONS} //animate to bottom left
      }),
      Animated.spring(this.state.pan, {
          ...SPRING_CONFIG,
          toValue: {x: width - SQUARE_DIMENSIONS, y: height - SQUARE_DIMENSIONS} // animated to bottom right
      }),
      Animated.spring(this.state.pan, {
            ...SPRING_CONFIG,
            toValue: {x: width - SQUARE_DIMENSIONS, y: 0} //animate to top right
      }),
      Animated.spring(this.state.pan, {
          ...SPRING_CONFIG,
          toValue: {x: 0, y: 0} // return to start
      })
    ]).start(cb);
  }
```

We just make a call that triggers the animation and calls itself on complete.



#Full/Live Code

## [https://rnplay.org/apps/QlPJ2Q](https://rnplay.org/apps/QlPJ2Q)

```
var React = require('react-native');
var Dimensions = require('Dimensions');

var {
  width,
  height
} = Dimensions.get('window');

var {
  AppRegistry,
  StyleSheet,
  View,
  Animated
} = React;

var SQUARE_DIMENSIONS = 30;
var SPRING_CONFIG = {tension: 2, friction: 3}; //Soft spring

var AnimatedSquare = React.createClass({
  getInitialState: function() {
    return {
        pan: new Animated.ValueXY()
    };
  },
  componentDidMount: function() {
    this.startAndRepeat();
  },
  startAndRepeat: function() {
    this.triggerAnimation(this.startAndRepeat);
  },
  triggerAnimation: function(cb) {
    Animated.sequence([
      Animated.spring(this.state.pan, {
            ...SPRING_CONFIG,
            toValue: {x: 0, y: height - SQUARE_DIMENSIONS} //animate to bottom left
      }),
      Animated.spring(this.state.pan, {
          ...SPRING_CONFIG,
          toValue: {x: width - SQUARE_DIMENSIONS, y: height - SQUARE_DIMENSIONS} // animated to bottom right
      }),
      Animated.spring(this.state.pan, {
            ...SPRING_CONFIG,
            toValue: {x: width - SQUARE_DIMENSIONS, y: 0} //animate to top right
      }),
      Animated.spring(this.state.pan, {
          ...SPRING_CONFIG,
          toValue: {x: 0, y: 0} // return to start
      })
    ]).start(cb);
  },
  getStyle: function() {
    return [
              styles.square, 
              {
                transform: this.state.pan.getTranslateTransform()
              }
            ];
  },
  render: function() {
    return (
      <View style={styles.container}>
        <Animated.View style={this.getStyle()} />
      </View>
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1
  },
  square: {
    width: SQUARE_DIMENSIONS,
    height: SQUARE_DIMENSIONS,
    backgroundColor: 'blue'
  } 
});

AppRegistry.registerComponent('AnimatedSquare', () => AnimatedSquare);

```
