---
layout: post
title: "React-Native Tinder like cards"
date: 2015-06-06 19:33
comments: true
categories: tinder react-native cards swipe react 
---

# CHECK OUT ====> [https://github.com/brentvatne/react-native-animated-demo-tinder](https://github.com/brentvatne/react-native-animated-demo-tinder) for a better demo with the new Animated API!!!

## Intro

The test of all good frameworks is how easy it is to implement Tinder right? Well with `react-native` we do get the benefits of flex box as well as some transforms which we'll take advantage of. 

We don't have access to an easy physics, even though they were added in IOS7. If you'd like physics you can use some JavaScript libraries like `rebound` [https://github.com/facebook/rebound-js](https://github.com/facebook/rebound-js) from Facebook, or any others that don't require a DOM.

What we'll make

{% img http://i.imgur.com/tTcT7xJ.png End Result %}

<!-- more -->


## Concept

We'll create a card. On touch press/grant we'll figure out the offset of the card to the touch and start generating the transform to move/rotate the card.

Well use the `style` `transform` property which we can find documentation here [https://facebook.github.io/react-native/docs/transforms.html#proptypes](https://facebook.github.io/react-native/docs/transforms.html#proptypes). However documentation is a little skimpy.

It's mostly straight forward once you dive in though.

## What we won't do.

Physics. You can implement a bouncy spring system, but we'll keep it simple with a drag concept.


## Create a basic card

We'll create a basic wrapper container and then create a card View.
We'll center everyting inside of our container using `alignItems` and `justifyContent` both `center`
Our card will just be `300` by `300`, with a little padding, and border.

```
render: function() {

    return (
      <View style={styles.container}>
          <View
            style={styles.card}
          >
          </View>
      </View>
    );
}

var styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center'
  },
  card: {
    borderWidth: 3,
    borderRadius: 3,
    borderColor: '#000',
    width: 300,
    height: 300,
    padding: 10
  }
  });

```

Now that we have a generic card we can make it look a little nicer with an image, and some text.


## Add an Image/Text to card


We'll add an image and set to a particular height. There is a current issue in `react-native` that doesn't maintain aspect ratio but that will be taken care of eventually.

We wrap our `Text` elements in `View` and position each `Text` item on the left and right.
There is a way to do this with flexbox but positioning like this is a little more explicit.

```
render: function() {

    return (
      <View style={styles.container}>
          <View
            style={styles.card}
          >
            <Image source={{uri: 'http://i.imgur.com/91AR0Lo.jpg'}} style={styles.cardImage} />
            <View>
              <Text style={styles.textLeft}>Rabbit, 10</Text>
              <Text style={styles.textRight}>1 Connection</Text>
            </View>
          </View>
      </View>
    );
}

var styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center'
  },
  card: {
    borderWidth: 3,
    borderRadius: 3,
    borderColor: '#000',
    width: 300,
    height: 300,
    padding: 10
  },
  cardImage: {
    height: 260,
  },
  textLeft: {
    position: 'absolute',
    left:0,
    top:0
  },
  textRight: {
    position: 'absolute',
    right: 0,
    top: 0
  }
});

```

## Things to know about style

Alright so there seems to be a lack of documentation around style in general. But `style` actually can receive an array.

You are capable of specifying a default layout, however doing overrides. For example take our card layout.

```
  card: {
    borderWidth: 3,
    borderRadius: 3,
    borderColor: '#000',
    width: 300,
    height: 300,
    padding: 10
  }

```

This gets applied, but what if at some point in time we wanted to change the `borderColor` based on `state`.
Well we just override it on the style attribute like so

```
    <View style={[styles.card, {borderColor: '#CCC'}]} />
```
So now the borderColor has a default but can be changed by just passing in an object.

This goes for `transform` as well which will set us up for the next challenge, actually dragging.


## Add in Drag

We'll use the gesture responder system. The robustness is great, however I was expecting a little more information like deltas over the course of each drag update. We aren't given that to my knowledge so we'll computer it ourselves.


How the gesture system works is it must ask each element that has a gesture responder if it should be allowed to drag or not. In our case we have one element and minimal logic so we'll just return true. However at any point you can cancel a gesture by returning false.

In our case you need to respond `true` to `onStartShouldSetResponder` and then each subsequent move `onMoveShouldSetResponder`. If those return true then it will call `onResponderMove` each time with the new event.

We'll use `_onStartShouldSetResponder` function to setup our initial drag. Each subsequent move we subtract and get the delta of the move.

```
  getInitialState: function() {
    return {
      x: 0,
      y: 0
    }
  },
  setPosition: function(e) {
    //Update our state with the deltaX/deltaY of the movement
    this.setState({
      x: this.state.x + (e.nativeEvent.pageX - this.drag.x),
      y: this.state.y + (e.nativeEvent.pageY - this.drag.y)
    });
    //Set our drag to be the new position so our delta can be calculated next time correctly
    this.drag.x = e.nativeEvent.pageX;
    this.drag.y = e.nativeEvent.pageY;
  },
  resetPosition: function(e) {
    this.dragging = false;
    //Reset on release
    this.setState({
      x: 0,
      y: 0,
    })
  },
  _onStartShouldSetResponder: function(e) {
    this.dragging = true;
    //Setup initial drag coordinates
    this.drag = {
      x: e.nativeEvent.pageX,
      y: e.nativeEvent.pageY
    }
    return true;
  },
  _onMoveShouldSetResponder: function(e) {
    return true;
  },
  getCardStyle: function() {
    var transform = [{translateX: this.state.x}, {translateY: this.state.y}];
    return {transform: transform};
  },
  render: function() {

    return (
      <View style={styles.container}>
          <View
            onResponderMove={this.setPosition}
            onResponderRelease={this.resetPosition}
            onStartShouldSetResponder={this._onStartShouldSetResponder}
            onMoveShouldSetResponder={this._onMoveShouldSetResponder}
            style={[styles.card, this.getCardStyle()]}
          >
            <Image source={{uri: 'http://i.imgur.com/91AR0Lo.jpg'}} style={styles.cardImage} />
            <View style={styles.cardTextContainer}>
              <Text style={styles.textLeft}>Rabbit, 10</Text>
              <Text style={styles.textRight}>1 Connection</Text>
            </View>
          </View>
      </View>
    );
  }
});
```

So now when a user press down on our card and starts dragging it'll move around. On release it'll snap back to position `0,0`.

You can see we use the `translateX` and `translateY` transform properties. These will cause the ability for the card to be dragged around but not have to make it position absolute.

## Add in Rotate

With Tinder and other card style systems as you drag the card left or right it will slightly rotate. It also rotates differently depending on the position you grab the card from (generally top or bottom).

The `transform` property on style also has a `rotate` option. This seems weird but it takes a string. That string can be something like `30deg` or `.05rad`. So it offers some flexibility. We'll use `degrees` since it's the easiest to comprehend.

We don't need to add anything to the view, just determine if we grabbed the card on the `top` or the `bottom`. Then depending on the offset drag make it rotate more as we move.

```
//Top of file
var Dimensions = require('Dimensions');
var windowSize = Dimensions.get('window');

//...
  _onStartShouldSetResponder: function(e) {
    this.dragging = true;

    this.rotateTop = e.nativeEvent.locationY <= 150;

    this.drag = {
      x: e.nativeEvent.pageX,
      y: e.nativeEvent.pageY
    }

    return true;
  },
  getRotationDegree: function(rotateTop, x) {
    var rotation = ( (x/windowSize.width) * 100)/3;

    var rotate = rotateTop ? 1 : -1,
        rotateString = (rotation * rotate) + 'deg';

    return rotateString;
  },
  getCardStyle: function() {
    var transform = [{translateX: this.state.x}, {translateY: this.state.y}];

    if (this.dragging) {
        transform.push({rotate: this.getRotationDegree(this.rotateTop, this.state.x)})
    }

    return {transform: transform};
  }
```

So we modify `_onStartShouldSetResponder` to determine wheter we grabbed top or bottom. We use the `locationY` property which is the point on the card that was touched. Since the card dimensions are `300x300` that means if the card was touched anywhere between `0 to 150` then it was touched on top.

Our `getCardStyle` will push a `rotate` object on if we are dragging.

We need to know how far around the screen you have dragged it from the center point.
So we get the screen dimensions, divide the width by the `pageX` coordinate which is just position of the element relative to the entire screen. To convert to degrees we multiply by `100` and divide by `3` to reduce the rotation.

If we touched on the bottom then we want to do a reverse rotation so we multiply by `-1`  and return a string that would return a value like `20.123deg` or `-20.123deg`.

## Add in Release Text

Great we have dragging, we have rotating. Now how do we know which way they let go?
Well we can use those window dimensions and the `pageX` movement to determine if the card was released on the left or right.

```
  resetPosition: function(e) {
    this.dragging = false;
    var left = e.nativeEvent.pageX < (windowSize.width/2),
        displayText = left ? 'Released left' : 'Released right';

    this.setState({
      x: 0,
      y: 0,
      lastDragDirectio: displayText
    })
  },
```


## Final Code

```
'use strict';

var React = require('react-native');
var Dimensions = require('Dimensions');
var windowSize = Dimensions.get('window');

var {
  StyleSheet,
  AppRegistry,
  Text,
  View,
  ActivityIndicatorIOS,
  Image,
  Navigator,
  TouchableOpacity,
  Animation
} = React;


var Application = React.createClass({
  getInitialState: function() {
    return {
      x: 0,
      y: 0,
      lastDragDirectio: 'Drag and Release'
    }
  },
  setPosition: function(e) {
    this.setState({
      x: this.state.x + (e.nativeEvent.pageX - this.drag.x),
      y: this.state.y + (e.nativeEvent.pageY - this.drag.y)
    });
    this.drag.x = e.nativeEvent.pageX;
    this.drag.y = e.nativeEvent.pageY;
  },
  resetPosition: function(e) {
    this.dragging = false;
    var left = e.nativeEvent.pageX < (windowSize.width/2),
        displayText = left ? 'Released left' : 'Released right';

    this.setState({
      x: 0,
      y: 0,
      lastDragDirectio: displayText
    })
  },
  getRotationDegree: function(rotateTop, x) {
    var rotation = ( (x/windowSize.width) * 100)/3;

    var rotate = rotateTop ? 1 : -1,
        rotateString = (rotation * rotate) + 'deg';

    return rotateString;
  },
  getCardStyle: function() {
    var transform = [{translateX: this.state.x}, {translateY: this.state.y}];

    if (this.dragging) {
        transform.push({rotate: this.getRotationDegree(this.rotateTop, this.state.x)})
    }

    return {transform: transform};
  },
  _onStartShouldSetResponder: function(e) {
    this.dragging = true;

    this.rotateTop = e.nativeEvent.locationY <= 150;

    this.drag = {
      x: e.nativeEvent.pageX,
      y: e.nativeEvent.pageY
    }

    return true;
  },
  _onMoveShouldSetResponder: function(e) {
    return true;
  },
  render: function() {

    return (
      <View style={styles.container}>
          <View
            onResponderMove={this.setPosition}
            onResponderRelease={this.resetPosition}
            onStartShouldSetResponder={this._onStartShouldSetResponder}
            onMoveShouldSetResponder={this._onMoveShouldSetResponder}
            style={[styles.card, this.getCardStyle()]}
          >
            <Image source={{uri: 'http://i.imgur.com/91AR0Lo.jpg'}} style={styles.cardImage} />
            <View style={styles.cardTextContainer}>
              <Text style={styles.textLeft}>Rabbit, 10</Text>
              <Text style={styles.textRight}>1 Connection</Text>
            </View>
          </View>
          <View style={styles.dragText}>
            <Text>{this.state.lastDragDirectio}</Text>
          </View>
      </View>
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center'
  },
  dragText: {
    position: 'absolute',
    bottom: 0,
    left: 0
  },
  card: {
    borderWidth: 3,
    borderRadius: 3,
    borderColor: '#000',
    width: 300,
    height: 300,
    padding: 10
  },
  cardImage: {
    height: 260,
  },
  textLeft: {
    position: 'absolute',
    left:0,
    top:0
  },
  textRight: {
    position: 'absolute',
    right: 0,
    top: 0
  }
});

module.exports = Application;
```

## Result

{% img http://i.imgur.com/q7siPyO.gif End Result %}

You can check out and play with the end result here.

## Preview Online!

Thanks to React Native Playground you can play with this code live online.

[https://rnplay.org/apps/6uPJug](https://rnplay.org/apps/6uPJug) 

Your homework can be to add a bounce when the card is released.