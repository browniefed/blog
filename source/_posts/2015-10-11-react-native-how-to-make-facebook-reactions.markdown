---
layout: post
title: "React Native - How to make Facebook Reactions"
date: 2015-10-11 11:35
comments: true
categories: react-native react facebook reactions gif animated
---

Facebook reactions is a new liking system that Facebook is trialing on a limited basis. Why wait for them to roll it out when we can roll it out ourselves

Thanks to [Engadget](http://www.engadget.com/2015/10/08/facebook-reactions/) who created a gif of the animations from a youtube video. After a little slicing up and conversion from white to transparent we have a nice bunch of terrible animated gifs... but they're animated so deal with it. Like/Love aren't transparent since imagmagick was destroying the white in them.

**This code will target .12 and above, running on .11 seemed to cause some slightly different layout issues. I've fixed them on RNPlay, so just a heads up. The difference is add `height:50` to our `likeContainer` **


**Update - Android:**

Animated Gifs are coming to React Native Android in .13. Also Android does not support `overflow: visible` so in order to get android to work we would need to get creative. On open they sit within the `overflow: hidden` container, once slid in we'd have to move them outside. Although they are gifs, so you'd have to find a way to sync em up. 

Maybe 2 sets, one slides in, the other is rendered but hidden, then you toggle which is hidden and slide the other up. Fun stuff!

## What are we building

{% img http://i.imgur.com/dFLU8SI.gif Neato %}

<!-- more --> 

## Concept

While holding down a button a pop up will show up, animated gifs will slide up from the bottom. While sliding and hovering over an item it will slide up slightly and scale up in size. On release of the finger, that item is selected, the images slide down and the pop up disappears.

This doesn't sound too bad, there are a few slight issues we'll run into that I'll call out. The main one is just the border around the container. We get creative with the component structure so the images will animate over the top of the border and not underneath it.

## Setup

{% raw %}


```
var React = require('react-native');
var {
  StyleSheet,
  Text,
  Image,
  View,
  PanResponder,
  TouchableOpacity,
  Animated
} = React;

var images = [
  {id: 'like', img: 'http://i.imgur.com/LwCYmcM.gif'},
  {id: 'love', img: 'http://i.imgur.com/k5jMsaH.gif'},
  {id: 'haha', img: 'http://i.imgur.com/f93vCxM.gif'},
  {id: 'yay', img: 'http://i.imgur.com/a44ke8c.gif'},
  {id: 'wow', img: 'http://i.imgur.com/9xTkN93.gif'},
  {id: 'sad', img: 'http://i.imgur.com/tFOrN5d.gif'},
  {id: 'angry', img: 'http://i.imgur.com/1MgcQg0.gif'}
]

var App = React.createClass({
  getInitialState: function() {
    return {
      selected: '',
      open: false
    };
  },
  componentWillMount: function() {
    this._imgLayouts = {};
    this._imageAnimations = {};
    this._scaleAnimation = new Animated.Value(0);

    images.forEach((img) => {
      this._imageAnimations[img.id] = {
        position: new Animated.Value(55),
        scale: new Animated.Value(1)
      };
    });

    this._panResponder = PanResponder.create({
      onStartShouldSetPanResponder: (evt, gestureState) => true,
      onStartShouldSetPanResponderCapture: (evt, gestureState) => true,
      onMoveShouldSetPanResponder: (evt, gestureState) => true,
      onMoveShouldSetPanResponderCapture: (evt, gestureState) => true,
    });
  },

  render: function() {
    return (
      <View style={styles.container}>
        <View
          style={styles.center}
          {...this._panResponder.panHandlers}
        >
          <Text>Like</Text>
          <Text>You selected: {this.state.selected}</Text>
          <Animated.View 
            style={[styles.likeContainer, this.getLikeContainerStyle()]}
          >
            <View style={styles.borderContainer} />
            <View style={styles.imgContainer}>
              {this.getImages()}
            </View>
          </Animated.View>
        </View>
      </View>
    );
  }
});

```

{% endraw %}

We setup our state, we have a `selected` which we will use to just display some selected text. Then we have an `open` which we set to false. I am not showing it yet, but we'll need to use `open` to dynamically control `overflow` for the container that holds our images so they can slide around correctly.

In our `componentWillMount` we setup some objects and create some animations. We'll need the coordinates of each image for selection purposes, we'll also have a map of our image animations, and we'll setup `_scaleAnimation` which we'll use to scale up the container when a user presses down.

Then we loop over each image, create a position, and scale animation for each. Why `55` and why not `ValueXY`? Well we are only animating the Y value so no need to have an extra animation. The `55` is our initial position, which we pass to `translateY` meaning, start this image `55` pixels down so we can eventually slide it up. 

Finally we'll create a PanResponder to handle users pressing.


## Handle Press and Open

So we'll put our `panHandlers` on our wrapping view so our coordinates are correct relatively for the images. That looks like this

```
       <View
          style={styles.center}
          {...this._panResponder.panHandlers}
        >
        ///Other views
        </View>
```

Then we need to adjust our PanResponder to call an open function on grant. Grant being the first thing that gets called when the touch is allowed.

```
this._panResponder = PanResponder.create({
      onStartShouldSetPanResponder: (evt, gestureState) => true,
      onStartShouldSetPanResponderCapture: (evt, gestureState) => true,
      onMoveShouldSetPanResponder: (evt, gestureState) => true,
      onMoveShouldSetPanResponderCapture: (evt, gestureState) => true,
      onPanResponderGrant: this.open
})
```

Lets adjust our `Animated.View` container.

```
  getLikeContainerStyle: function() {
    return {
            transform: [{scaleY: this._scaleAnimation}],
            overflow: this.state.open ? 'visible': 'hidden'
          };
  },

  //other render components
  <Animated.View 
    style={[styles.likeContainer, this.getLikeContainerStyle()]}
  >
    <View style={styles.borderContainer} />
    <View style={styles.imgContainer}>
      {this.getImages()}
    </View>
  </Animated.View>
```

Now our open function.

```
  open: function() {
    Animated.parallel([
      Animated.timing(this._scaleAnimation, {
        duration: 100,
        toValue: 1
      }),
      Animated.stagger(50, this.getImageAnimationArray(0))
    ]).start(() => this.setState({open: true}));
  },
  getImageAnimationArray: function(toValue) {
    return images.map((img) => {
      return Animated.timing(this._imageAnimations[img.id].position, {
        duration: 200,
        toValue: toValue
      })
    });
  },
```
We'll queue up some `parallel` animations so they run at the same time. The first is the initial scaling of the container. We'll do it over `100ms`, and go from 0 to 1.
The next thing we'll do is use `Animated.stagger`, this will trigger an array of animations, but with a delay in between each one. That means we can easily slide up each image reaction separately with `50ms` inbetween. What a handy function.

So we'll do a little forward thinking and create a `getImageAnimationArray` function because we'll have to do the exact opposite for the `close` so lets make a reusable function.

Finally we'll `setState` AFTER the animation is complete. This will not cause a jitter in our animation. This also allows the container to hide the images to start so they slide up. Then after the container is open we set `overflow: visible` so we can scale and slide them up and they'll actually be visible. If we don't do this then the images won't be able visible when the user tries to select them.

## Handle release and close

Now modify the `_panResponder` for the release to close it, we basically just do the exact opposite of open.

```
      onPanResponderRelease: (evt, gestureState) => {
         this.close()
      }
```

And our close function

```
  close: function(cb) {
    this.setState({open: false}, () => {
      Animated.stagger(100,[
        Animated.parallel(this.getImageAnimationArray(55, 0).reverse() ),
        Animated.timing(this._scaleAnimation, {
          duration: 100,
          toValue: 0
        })
      ]).start(cb);
    })
  },
```
We need to do a setState to make the containers overflow hidden. This is tricky because it'll instantly cause anything selected to be hidden. So we'll need to account for this in our release function later. But at the moment, we set `open` to false. 

Rather than parallel we'll stagger our 2 animations. Our first will start and cause our reaction gifs to all slide down at the same time. Then `100ms` later we'll scale the container down to 0. This just adds a slight disappearing effect.


## Calculate image positions

Before we can do image selections we need to know where our images coordinates are. Luckily we can do that with the `onLayout` callback.

{% raw %}

```
  getImages: function() {
    return images.map((img) => {
      return (
        <Animated.Image 
          onLayout={this.handleLayoutPosition.bind(this, img.id)}
          key={img.id} 
          source={{uri: img.img}} 
          style={[
              styles.img,
              {
                transform: [
                  {scale: this._imageAnimations[img.id].scale},
                  {translateY: this._imageAnimations[img.id].position}
                ]
              }
          ]} 
        />
      );
    })
  },
```
{% endraw %}

This is our image rendering function. It iterates over our images, passes in all necesary info and binds our `onLayout` function with the `image.id` which is just the reaction name. You can see here we also pass in our transform animations for `scale` and `translateY`

```
  handleLayoutPosition: function(img, position) {
    this._imgLayouts[img] = {
      left: position.nativeEvent.layout.x,
      right: position.nativeEvent.layout.x + position.nativeEvent.layout.width
    }
  },
```
Here we setup a map, we'll set the `left` which is just the relative `x` coordinate, then setup our `right` which is `x + width`.


## Add in hover abilities

```
      onPanResponderMove: (evt, gestureState) => {
        var hoveredImg = this.getHoveredImg(Math.ceil(evt.nativeEvent.locationX));

        if (hoveredImg && this._hoveredImg !== hoveredImg) {
          this.animateSelected(this._imageAnimations[hoveredImg])
        }
        if (this._hoveredImg !== hoveredImg && this._hoveredImg) {
          this.animateFromSelect(this._imageAnimations[this._hoveredImg]);
        }

        this._hoveredImg = hoveredImg;
      },
```

We just do some basic logic here. If we have a `hoveredImg` and it's different than before, then call the animate function to make the image slide and scale up.
Also if we have a current `this._hoveredImg` and it's different then that means there is a new hovered image, so lets animate the previous image and scale it back down.

```
  animateSelected: function(imgAnimations) {
    Animated.parallel([
      Animated.timing(imgAnimations.position, {
        duration: 150,
        toValue: -30
      }),
      Animated.timing(imgAnimations.scale, {
        duration: 150,
        toValue: 1.8
      })
    ]).start();
  },
  animateFromSelect: function(imgAnimations, cb) {
    Animated.parallel([
      Animated.timing(imgAnimations.position, {
        duration: 50,
        toValue: 0
      }),
      Animated.timing(imgAnimations.scale, {
        duration: 50,
        toValue: 1
      })
    ]).start(cb);
  },
  getHoveredImg: function(x) {
    return Object.keys(this._imgLayouts).find((key) => {
      return x >= this._imgLayouts[key].left && x <= this._imgLayouts[key].right;
    })
  },
```

Nothing too much here to call out. Our `getHoveredImg` function runs through our layouts, and just checks if the `x` coordinate of the finger press is between the `left` and `right`. If it finds one it returns the reaction id.

The `animateSelected` and `animatedFromSelect` both take one the image animation objects and applies different animations to each.

## Add in selection text

```
      onPanResponderRelease: (evt, gestureState) => {
        if (this._hoveredImg) {
          this.animateFromSelect(this._imageAnimations[this._hoveredImg], this.close.bind(this, this.afterClose) )
        } else {
          this.close(this.afterClose);
        }
      }
```

I had mentioned this before. On the release if we have a current selection, then we need to first animate the selected image back to it's original position, which we have `animateFromSelect` already. Then when that is complete the callback is called which is our close function, which has a callback to trigger after close.

```
  afterClose: function() {
    if (this._hoveredImg) {
      this.setState({
        selected: this._hoveredImg
      })
    }

    this._hoveredImg = '';
  },
```

Our after close just sets some text of the hovered img and clears it. This may not be necessary for an actual app, but this is just so I can have some text to show a user they selected a reaction.

## Weird setup and styles

I'll just point out some "weird" things I'm doing to make things work correctly. Mainly rather than adding in the border on the `likeContainer` we need to add in 2 separate containers.

Our element structure

```
<Animated.View 
            style={[styles.likeContainer, this.getLikeContainerStyle()]}
          >
            <View style={styles.borderContainer} />
            <View style={styles.imgContainer}>
              {this.getImages()}
            </View>
          </Animated.View>
```
Our weird styling.

```
  likeContainer: {
    position: 'absolute',
    left: -10,
    top: -30,
    padding: 5,
    flex: 1,
    backgroundColor: '#FFF',
    borderColor: 'transparent',
    borderWidth: 0,
    borderRadius: 20,
  },
  borderContainer: {
    backgroundColor: 'transparent',
    position: 'absolute',
    left: 0,
    right: 0,
    top: 0,
    bottom: 0,
    borderWidth: 1,
    borderColor: '#444',
    borderRadius: 20
  },
  imgContainer: {
    backgroundColor: 'transparent',
    flexDirection: 'row',
  },
```

The `borderContainer` is absolutely positioned inside of the `likeContainer`. The `borderContainer` is just that, a bunch of border data.

The `likeContainer` also has border radius, transparent border styling so that it is shaped perfectly for overflow when the images slide down.


## DONE

And there we have it. This is by no means an exact replica. It needs some serious polish and also needs to add a delay before opening when the users presses. It's good enough for me though. So go forth and add reactions to your app!

I didn't do anything crazy and assumed it would work on android. There a few issues with it so I'm not sure exactly what's going on.


#### Check it out live on RNPlay [https://rnplay.org/apps/lIVMng](https://rnplay.org/apps/lIVMng)

As stated before there are minor tweaks from the code to make it work with react .11, not entirely certain why there are differences but there are.

{% img http://i.imgur.com/dFLU8SI.gif Neato %}


Full code here

{% raw %}

```
var React = require('react-native');
var {
  StyleSheet,
  Text,
  Image,
  View,
  PanResponder,
  TouchableOpacity,
  Animated
} = React;

var images = [
  {id: 'like', img: 'http://i.imgur.com/LwCYmcM.gif'},
  {id: 'love', img: 'http://i.imgur.com/k5jMsaH.gif'},
  {id: 'haha', img: 'http://i.imgur.com/f93vCxM.gif'},
  {id: 'yay', img: 'http://i.imgur.com/a44ke8c.gif'},
  {id: 'wow', img: 'http://i.imgur.com/9xTkN93.gif'},
  {id: 'sad', img: 'http://i.imgur.com/tFOrN5d.gif'},
  {id: 'angry', img: 'http://i.imgur.com/1MgcQg0.gif'}
]

var App = React.createClass({
  getInitialState: function() {
    return {
      selected: '',
      open: false
    };
  },
  componentWillMount: function() {
    this._imgLayouts = {};
    this._imageAnimations = {};
    this._hoveredImg = '';

    this._scaleAnimation = new Animated.Value(0);

    images.forEach((img) => {
      this._imageAnimations[img.id] = {
        position: new Animated.Value(55),
        scale: new Animated.Value(1)
      };
    })

    this._panResponder = PanResponder.create({
      onStartShouldSetPanResponder: (evt, gestureState) => true,
      onStartShouldSetPanResponderCapture: (evt, gestureState) => true,
      onMoveShouldSetPanResponder: (evt, gestureState) => true,
      onMoveShouldSetPanResponderCapture: (evt, gestureState) => true,
      onPanResponderGrant: this.open,
      onPanResponderMove: (evt, gestureState) => {
        var hoveredImg = this.getHoveredImg(Math.ceil(evt.nativeEvent.locationX));

        if (hoveredImg && this._hoveredImg !== hoveredImg) {
          this.animateSelected(this._imageAnimations[hoveredImg])
        }
        if (this._hoveredImg !== hoveredImg && this._hoveredImg) {
          this.animateFromSelect(this._imageAnimations[this._hoveredImg]);
        }

        this._hoveredImg = hoveredImg;
      },
      onPanResponderRelease: (evt, gestureState) => {
        if (this._hoveredImg) {
          this.animateFromSelect(this._imageAnimations[this._hoveredImg], this.close.bind(this, this.afterClose) )
        } else {
          this.close(this.afterClose);
        }
      }
    });
  },
  afterClose: function() {
    if (this._hoveredImg) {
      this.setState({
        selected: this._hoveredImg
      })
    }

    this._hoveredImg = '';
  },
  animateSelected: function(imgAnimations) {
    Animated.parallel([
      Animated.timing(imgAnimations.position, {
        duration: 150,
        toValue: -30
      }),
      Animated.timing(imgAnimations.scale, {
        duration: 150,
        toValue: 1.8
      })
    ]).start();
  },
  animateFromSelect: function(imgAnimations, cb) {
    Animated.parallel([
      Animated.timing(imgAnimations.position, {
        duration: 50,
        toValue: 0
      }),
      Animated.timing(imgAnimations.scale, {
        duration: 50,
        toValue: 1
      })
    ]).start(cb);
  },
  getHoveredImg: function(x) {
    return Object.keys(this._imgLayouts).find((key) => {
      return x >= this._imgLayouts[key].left && x <= this._imgLayouts[key].right;
    })
  },

  getImageAnimationArray: function(toValue) {
    return images.map((img) => {
      return Animated.timing(this._imageAnimations[img.id].position, {
        duration: 200,
        toValue: toValue
      })
    });
  },
  open: function() {
    Animated.parallel([
      Animated.timing(this._scaleAnimation, {
        duration: 100,
        toValue: 1
      }),
      Animated.stagger(50, this.getImageAnimationArray(0))
    ]).start(() => this.setState({open: true}));
  },
  close: function(cb) {
    this.setState({open: false}, () => {
      Animated.stagger(100,[
        Animated.parallel(this.getImageAnimationArray(55, 0).reverse() ),
        Animated.timing(this._scaleAnimation, {
          duration: 100,
          toValue: 0
        })
      ]).start(cb);
    })
  },
  handleLayoutPosition: function(img, position) {
    this._imgLayouts[img] = {
      left: position.nativeEvent.layout.x,
      right: position.nativeEvent.layout.x + position.nativeEvent.layout.width
    }
  },
  getImages: function() {
    return images.map((img) => {
      return (
        <Animated.Image 
          onLayout={this.handleLayoutPosition.bind(this, img.id)}
          key={img.id} 
          source={{uri: img.img}} 
          style={[
              styles.img,
              {
                transform: [
                  {scale: this._imageAnimations[img.id].scale},
                  {translateY: this._imageAnimations[img.id].position}
                ]
              }
          ]} 
        />
      );
    })
  },
  getLikeContainerStyle: function() {
    return {
            transform: [{scaleY: this._scaleAnimation}],
            overflow: this.state.open ? 'visible': 'hidden',
          };
  },
  render: function() {
    return (
      <View style={styles.container}>
        <View
          style={styles.center}
          {...this._panResponder.panHandlers}
        >
          <Text>Like</Text>
          <Text>You selected: {this.state.selected}</Text>
          <Animated.View 
            style={[styles.likeContainer, this.getLikeContainerStyle()]}
          >
            <View style={styles.borderContainer} />
            <View style={styles.imgContainer}>
              {this.getImages()}
            </View>
          </Animated.View>
        </View>
      </View>
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  center: {
    position: 'absolute',
    left: 50,
    top: 300
  },
  likeContainer: {
    position: 'absolute',
    left: -10,
    top: -30,
    padding: 5,
    flex: 1,
    backgroundColor: '#FFF',
    borderColor: 'transparent',
    borderWidth: 0,
    borderRadius: 20,
  },
  borderContainer: {
    backgroundColor: 'transparent',
    position: 'absolute',
    left: 0,
    right: 0,
    top: 0,
    bottom: 0,
    borderWidth: 1,
    borderColor: '#444',
    borderRadius: 20
  },
  imgContainer: {
    backgroundColor: 'transparent',
    flexDirection: 'row',
  },
  img: {
    marginLeft: 5,
    marginRight: 5,
    width: 30,
    height: 30,
    overflow: 'visible'
  }
});

module.exports = App;
```

{% endraw %}