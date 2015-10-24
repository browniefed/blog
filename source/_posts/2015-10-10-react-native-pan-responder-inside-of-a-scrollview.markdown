---
layout: post
title: "React Native - Pan Responder inside of a ScrollView"
date: 2015-10-10 06:59
comments: true
categories: react-native panresponder scrollview
---

Lets talk `PanResponder` in a `ScrollView`. This gets brought up frequently, so lets address it. 

## Scenario

You've got a `PanResponder` in your `ScrollView`. When you scroll you want it to scroll, when you interact with the component with the `PanResponder` you want it to do `PanResponder` things. 

## What happens

Things start scrolling just fine. You attemp to drag, all goes swimmingly, then the `ScrollView` scrolls, your drag stops working and it just sits there stuck until you go re-interact with it. Yikes. You curse the react native gods and begrudgingly start learning Objective-C.

## Janky

{% img http://i.imgur.com/2d8nB6u.gif Screw it, lets just go native %}

<!-- more -->

## Solution

The magic solution is `scrollEnabled={false}`. That's it. Seriously. Sadly it's only supported on `ios` as of me writing this blog post. I'm sure it'll be supported in the future for `android`.

{% img http://i.imgur.com/Z34hsmN.gif Sort of acceptable %}

## Done

#### Play with it here [https://rnplay.org/apps/we3HnA](https://rnplay.org/apps/we3HnA)


## Full Code Here

{% raw %}


```
'use strict';

var React = require('react-native');
var {
  AppRegistry,
  StyleSheet,
  PanResponder,
  ScrollView,
  View,
  Animated,
  Text
} = React;

var SampleApp = React.createClass({
  getInitialState: function() {
        return {
      scroll: true,
      pan: new Animated.ValueXY()
    }
  },
  componentWillMount: function() {
    this._panResponder = PanResponder.create({
      onStartShouldSetPanResponder: (evt, gestureState) => true,
      onStartShouldSetPanResponderCapture: (evt, gestureState) => true,
      onMoveShouldSetPanResponder: (evt, gestureState) => true,
      onMoveShouldSetPanResponderCapture: (evt, gestureState) => true,
      onPanResponderGrant: () => this.setState({scroll: false}),
      onPanResponderMove: Animated.event([null, {dx: this.state.pan.x, dy: this.state.pan.y}]),
        onPanResponderRelease: () => this.setState({scroll: true})
    })
  },
  render: function() {
    return (
      <View style={styles.container}>
                <ScrollView 
            style={{flex: 1}}
          scrollEnabled={this.state.scroll}                      
        >
            <Animated.View 
                style={{transform: this.state.pan.getTranslateTransform(), position: 'absolute', left: 150, top: 150}}
                {...this._panResponder.panHandlers}
            >
                <Text>Drag Me</Text>
            </Animated.View>
        </ScrollView>
      </View>
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1,

  },
});

AppRegistry.registerComponent('SampleApp', () => SampleApp);
```

{% endraw %}