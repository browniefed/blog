---
layout: post
title: "React Native Youtube Animated Video Slide"
date: 2015-08-31 08:58
comments: true
categories: react-native react native youtube video slider animation animated
---

# Introduciton

I was going to spend some time digging into [gl-react-native](https://github.com/ProjectSeptemberInc/gl-react-native) when someone asked in the slack channel how to achieve youtube like video sliding navigation. Now I won't get into the navigation portion, just the animation part. The navigation portion can be solved by basically treating video routes as a modal. Thus they are always over the top of whatever the previous route was.

# What are we building?

If you haven't seen it, YouTube allows you to drag the current playing video down to the bottom right corner and have it continue to play while you browse the rest of the app.

Something like this.

{% img http://i.imgur.com/gwbkw5f.gif YouTube demo clone %}

<!-- more -->

# Setup

React native doesn't have a `Video` component but Brent Vatne created a fantastic video component called [react-native-video](https://github.com/brentvatne/react-native-video). This allows you to use videos added to your app bundle, or external video urls.

So unlike a normal project you will need to run `npm install react-native-video --save`. Then follow the instructions on the `README` in `react-native-video` on how to add the library in XCode.

```
var React = require('react-native');
var Video = require('react-native-video');

var Dimensions = require('Dimensions');
var {width: deviceWidth, height: deviceHeight} = Dimensions.get('window');


var videoWidth = deviceWidth,
    videoHeight = Math.round((deviceWidth/16)*9);

var {
  AppRegistry,
  StyleSheet,
  View,
  Text,
  Animated, 
  PanResponder,
  ScrollView
} = React;


var AnimatedVideo = Animated.createAnimatedComponent(Video);
var AnimatedScrollView = Animated.createAnimatedComponent(ScrollView);

```

So a few things to call out here.

We pull in dimensions of the screen and name them `deviceHeight` and `deviceWidth`. This is so we can calculate `videoWidth` and `videoHeight`. We use a simple calculation to create a video that is in the `16:9` aspect ratio.

Then we use `createAnimatedComponent` to create an animated `ScrollView` and `AnimatedVideo` element.


# Basic Component

```
var YoutubeVideoSlide = React.createClass({
  getInitialState: function() {
    return {
      scale: new Animated.Value(1),
      position: new Animated.ValueXY(),
    };
  },
  render: function() {
    return (
      <View style={styles.container}>
              <AnimatedVideo 
                  source={{uri: "http://clips.vorwaerts-gmbh.de/big_buck_bunny.mp4"}}
                  style={styles.videoSizing}
                  rate={1}
                  paused={false}
                  volume={1}
                  resizeMode={'stretch'}
                  repeat={true} 
              />
            <AnimatedScrollView style={styles.container}>
              <View style={styles.comment}>
                <Text>Video Comment</Text>
              </View>
            </AnimatedScrollView>
      </View>
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1
  },
  videoSizing: {
    width: videoWidth,
    height: videoHeight
  },
  comment: {
    height: 100
  }
});

```

We setup `flex:1` on the `View` so that the `ScrollView` will size correctly. We set the `width` and `height` on the `AnimatedVideo` to our `16:9` ratio. In the state we setup the initial scale to `1`, and an `X,Y` position for the moving video.

# Setup the PanResponder

```
    this._panResponder = PanResponder.create({
      onStartShouldSetPanResponder: (evt, gestureState) => true,
      onStartShouldSetPanResponderCapture: (evt, gestureState) => true,
      onMoveShouldSetPanResponder: (evt, gestureState) => true,
      onMoveShouldSetPanResponderCapture: (evt, gestureState) => true,
      onPanResponderMove: Animated.event([
          null, 
          {
              dy: this.state.position.y
          }
      ])
    });
```

We'll do the normal "say yes to allowing us to touch things", and then we'll setup an `Animated.event`. This is a helper function to traverse the arguments from `onPanResponderMove` and update an Animated value.

`onPanResponderMove` gets called with an `event` as the first argument, and `gestureState` as the second. The `null` tells `Animated.event` to ignore the first argument, but to grab the `dy` from the `gestureState` and update the `y` value on our `Animated.ValueXY` we created.


# Interpolate and Animate

```
    this._scale = this.state.position.y.interpolate({
      inputRange: [0, deviceHeight ],
      outputRange: [1, .71],
      extrapolate: 'clamp'
    });

    this._translateY = this.state.position.y.interpolate({
      inputRange: [0, deviceHeight],
      outputRange: [0, deviceHeight],
      extrapolate: 'clamp'
    });

    this.state.position.y.addListener((value) => {
      this._y = value.value;
      var scaleValue = this._scale.getAnimatedValue();
      var currentVideoWidth = scaleValue * videoWidth;
      var buffer = ((videoWidth - currentVideoWidth)/2);
      this.state.position.x.setValue(buffer);
    }.bind(this));

    this._opacity = this.state.position.y.interpolate({
      inputRange: [0, deviceHeight ],
      outputRange: [1, .1]
    });
```

Now we need to setup a few more things in the `componentWillMount`. The `this._scale` will be set on a scale transform. This is directly tied to the `y` position of us dragging. So we setup an interpolate, which goes from `0` to the `deviceHeight` and map that to an output range of `1` to `.71`. The `.71` was a trial and error number, I'm sure there is math to calculate this but I guessed till I got it right.

The `clamp` is to say that this value cannot go above or below these values.

You may be wondering why we setup `translateY` when the `inputRange` and `outputRange` are exactly the same. The `clamp` is the key part. This means that if a user tries to slide the video up it will not grow in scale, or slide the video upwards. It can only go down.

We setup the listener so we can keep track of our `y` value, we'll get into that later. We'll use `getAnimatedValue` to get the current scale. **THIS WILL NOT WORK IN .11-rc** and above. You cannot currently listen on interpolated values, and we need the interpolated value for our math.

We get the original `videoWidth` and multiply it times the scale value so we can get the current `videoWidth`. We subtract the `currentVideoWidth` from the `videoWidth` and divide it by `2` to get the current buffer. That buffer is the space between the right side of the video and the current scaled video. If we don't do this then the X/Y scaling on the video will just squish it to the middle of the screen. What we want to do is have it slide down the right side of the screen.

Finally we setup the opacity, this is for the `scrollView` to slowly fade as we swipe down.

# Setup the styling

```
  getScalePosition: function() {
    return {
      transform: [
        {scale: this._scale},
        {translateX: this.state.position.x},
        {translateY: this._translateY}
      ]
    }
  },
  getScrollOffset: function() {
    return {
      transform: [
        {translateY: this._translateY},
      ],
      opacity: this._opacity
    }
  },

    <AnimatedVideo 
      {...this._panResponder.panHandlers}
      source={{uri: "http://clips.vorwaerts-gmbh.de/big_buck_bunny.mp4"}}
      style={[styles.videoSizing, this.getScalePosition()]}
      rate={this.state.rate}
      paused={this.state.paused}
      volume={this.state.volume}
      muted={this.state.muted}
      resizeMode={this.state.resizeMode}
      repeat={true}
    />
    <AnimatedScrollView style={[styles.container, this.getScrollOffset()]}>
    </AnimatedScrollView>
```

Here we set default styling, and then make our calls to get the style objects with our animated values. For our video scale position we pass our `this._scale` to `scale` to transform both `scaleX`, and `scaleY`. We pass in our position x, and our `this._translateY` that is clamped.

For our `ScrollView` , we setup `translateY`, and pass in the opacity we created.


# Set Offset and Animate the Release

```
    this._panResponder = PanResponder.create({
      onStartShouldSetPanResponder: (evt, gestureState) => true,
      onStartShouldSetPanResponderCapture: (evt, gestureState) => true,
      onMoveShouldSetPanResponder: (evt, gestureState) => true,
      onMoveShouldSetPanResponderCapture: (evt, gestureState) => true,
      onPanResponderGrant: function() {
        this.state.position.y.setOffset(this._y)
      }.bind(this),
      onPanResponderMove: Animated.event([
          null, 
          {
              dy: this.state.position.y
          }
      ]),
      onPanResponderRelease: (e, gestureState) => {
        this.state.position.flattenOffset();

        if (gestureState.dy >= 40) {
          Animated.timing(this.state.position.y, {
            duration: 200,
            toValue: deviceHeight
          }).start();
        } else {
          Animated.timing(this.state.position.y, {
            duration: 200,
            toValue: 0
          }).start();
        }

      }.bind(this)
    });
```

When I said we'd get to why we kept track of the `y` this is why. This is so we can set the offset of the initial press and prevent the video from jumping. It will work for sliding down, but with `dy` it is always the delta. So on the slide up from the bottom it would immediately jump to the top, and we don't want that.

```
      onPanResponderGrant: function() {
        this.state.position.y.setOffset(this._y)
      }.bind(this),
```
This code sets the initial offset when you first touch to whatever the current y value is.

```
      onPanResponderRelease: (e, gestureState) => {
        this.state.position.flattenOffset();

        if (gestureState.dy >= 100) {
          Animated.timing(this.state.position.y, {
            duration: 200,
            toValue: deviceHeight
          }).start();
        } else {
          Animated.timing(this.state.position.y, {
            duration: 200,
            toValue: 0
          }).start();
        }

      }.bind(this)
```

On release we flatten the offset. Which just squashes the current value and offset together.
If the user has moved greater than `100` pixels then we'll animate the video to the bottom of the screen. If not we'll animate it back to the top.


# Final

As always check it out on RNPlay at [https://rnplay.org/apps/Cp_SSA](https://rnplay.org/apps/Cp_SSA). 
The video source comes from [http://camendesign.com/code/video_for_everybody/test.html](http://camendesign.com/code/video_for_everybody/test.html). It's a wonderfully open sourced mp4 video that we can link to test out.

# Full Code

```
var React = require('react-native');
var Dimensions = require('Dimensions');
var {width: deviceWidth, height: deviceHeight} = Dimensions.get('window');
var Video = require('react-native-video');


var videoWidth = deviceWidth,
    videoHeight = Math.round((deviceWidth/16)*9);

var {
  AppRegistry,
  StyleSheet,
  View,
  Text,
  Animated, 
  PanResponder,
  ScrollView
} = React;


var AnimatedVideo = Animated.createAnimatedComponent(Video);
var AnimatedScrollView = Animated.createAnimatedComponent(ScrollView);

var YoutubeVideoSlide = React.createClass({
  getInitialState: function() {
    return {
      rate: 1,
      volume: 1,
      muted: true,
      resizeMode: 'stretch',
      duration: 0.0,
      currentTime: 0.0,
      scale: new Animated.Value(1),
      position: new Animated.ValueXY(),

    };
  },
  _y: 0,
  componentWillMount: function() {
    this._panResponder = PanResponder.create({
      onStartShouldSetPanResponder: (evt, gestureState) => true,
      onStartShouldSetPanResponderCapture: (evt, gestureState) => true,
      onMoveShouldSetPanResponder: (evt, gestureState) => true,
      onMoveShouldSetPanResponderCapture: (evt, gestureState) => true,
      onPanResponderGrant: function() {
        this.state.position.y.setOffset(this._y)
      }.bind(this),
      onPanResponderMove: Animated.event([
          null, 
          {
              dy: this.state.position.y
          }
      ]),
      onPanResponderRelease: (e, gestureState) => {
        this.state.position.flattenOffset();

        if (gestureState.dy >= 100) {
          Animated.timing(this.state.position.y, {
            duration: 200,
            toValue: deviceHeight
          }).start();
        } else {
          Animated.timing(this.state.position.y, {
            duration: 200,
            toValue: 0
          }).start();
        }

      }.bind(this)
    });

    this._scale = this.state.position.y.interpolate({
      inputRange: [0, deviceHeight ],
      outputRange: [1, .71],
      extrapolate: 'clamp'
    });

    this._translateY = this.state.position.y.interpolate({
      inputRange: [0, deviceHeight],
      outputRange: [0, deviceHeight],
      extrapolate: 'clamp'
    });

    this.state.position.y.addListener((value) => {
      this._y = value.value;
      var scaleValue = this._scale.getAnimatedValue();
      var currentVideoWidth = scaleValue * videoWidth;
      var buffer = ((videoWidth - currentVideoWidth)/2);
      this.state.position.x.setValue(buffer);
    }.bind(this));

    this._opacity = this.state.position.y.interpolate({
      inputRange: [0, deviceHeight ],
      outputRange: [1, .1]
    });

  },
  getScalePosition: function() {
    return {
      transform: [
        {scale: this._scale},
        {translateX: this.state.position.x},
        {translateY: this._translateY}
      ]
    }
  },
  getScrollOffset: function() {
    return {
      transform: [
        {translateY: this._translateY},
      ],
      opacity: this._opacity
    }
  },
  render: function() {
    return (
      <View style={styles.container}>
              <AnimatedVideo 
                  {...this._panResponder.panHandlers}
                  source={{uri: "http://clips.vorwaerts-gmbh.de/big_buck_bunny.mp4"}}
                  style={[styles.videoSizing, this.getScalePosition()]}
                  rate={this.state.rate}
                  paused={this.state.paused}
                  volume={this.state.volume}
                  muted={this.state.muted}
                  resizeMode={this.state.resizeMode}
                  repeat={true} 
              />
            <AnimatedScrollView style={[styles.container, this.getScrollOffset()]}>
              <View style={styles.comment}>
                <Text>Video Comment</Text>
              </View>
              <View style={styles.comment}>
                <Text>Video Comment</Text>
              </View>
              <View style={styles.comment}>
                <Text>Video Comment</Text>
              </View>
              <View style={styles.comment}>
                <Text>Video Comment</Text>
              </View>
              <View style={styles.comment}>
                <Text>Video Comment</Text>
              </View>
              <View style={styles.comment}>
                <Text>Video Comment</Text>
              </View>
              <View style={styles.comment}>
                <Text>Video Comment</Text>
              </View>
            </AnimatedScrollView>
      </View>
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1
  },
  videoSizing: {
    width: videoWidth,
    height: videoHeight
  },
  comment: {
    height: 100
  }
});


```

