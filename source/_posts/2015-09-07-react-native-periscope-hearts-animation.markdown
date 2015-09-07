---
layout: post
title: "React Native Periscope Hearts Animation"
date: 2015-09-07 11:02
comments: true
categories: react react-native periscope hearts animation
---

# Introduction

I was contacted asking if it was possible to recreate the periscope heart animations in react-native. I was also linked to someone rebuilding the same effect in Framer.js, you can check out the linked video here [https://www.youtube.com/watch?v=qFUXxqzZytU](https://www.youtube.com/watch?v=qFUXxqzZytU).

Periscope not only has an iPhone/Android app but it also has a web app with the same heart effect. I could take a look at the animations being done on it but we'll just eye ball it.

# What we are building

{% img http://i.imgur.com/5JhgzQV.gif Purple floating fading swaying rotating hearts %}

### Live code: [https://rnplay.org/apps/8VhSjw](https://rnplay.org/apps/8VhSjw)


<!-- more -->

# What it looks like on Periscope

If you don't know what I'm talking about, the heart animation looks like this. 

{% img http://i.imgur.com/sEJf9Md.gif Periscope hearts %}

# Break Down

In order to replicate the animation we have to break it down into it's parts. 

* Heart appears and scales up then quickly scales back down. Animation `Scale 0 => Scale 1.2 => Scale 1`
* Heart moves upwards with a slight sway to the left then back to the right. `X/Y from 0 => deviceHeight/2`
* Slight rotation of the heart through out each sway `rotate -15deg => 0 => 15deg`
* Heart opacity fades out over entire animation `opacity 1 => 0`

We likely won't need `Animated` values for all of these, the goal of Animating is to attempt to interpolate other values from one `Animated.Value`. In our case that one constant is the `X/Y` values. All of the other animations are dependent on where the heart is currently located.

The opacity is derived from the `X/Y` that it can be an interpolated value. The scale could be interpolated too, with 3 small input ranges, and output ranges of `[0, 1.2, 1]`. The rotation can also be interpolated based upon the `X` value. Even the X can be interpolated based upon the Y if we determine that we want 2 sways to happen before the animation is complete.

# Make the Heart

Now we have to decide how to make the heart. One option is to use an `<Image />` however this means I have to open up an image editor and I'm a developer, not a designer! If you haven't seen [The Shapes of CSS](https://css-tricks.com/examples/ShapesOfCSS/) I recommend checking it out. For basics shapes CSS will work great, and in our case a heart is a simple shape. 

It's composed of 2 objects overlayed on top of each other. Two squares, with top left / top right border radiuses and then drop them on top of each other.

Apart they look like this

{% img http://i.imgur.com/mUu1Yt8.png Two squares not slammed together %}

Then together we get a heart!

{% img http://i.imgur.com/o8WOlDU.png Now it's a heart %}

This also gives us control over the color more easily, the sizing, and anything else you can do with a simple view.


# Setup

```
var React = require('react-native');
var Dimensions = require('Dimensions');

var {
  width: deviceWidth,
  height: deviceHeight
} = Dimensions.get('window');

var {
  AppRegistry,
  StyleSheet,
  View,
  Animated,
  TouchableWithoutFeedback
} = React;

var ANIMATION_END_Y = Math.ceil(deviceHeight * .5);
var NEGATIVE_END_Y = ANIMATION_END_Y * -1;
var startCount = 1;
```
We'll bring in the necessary includes. The `ANIMATION_END_Y` and the reverse `NEGATIVE_END_Y` will become apparent as what they are later. Due to some interpolation we'll need to do some trickery to make our animation interpolations make more sense. 

# Create The Heart

As we showed before the heart is 2 pieces. These pieces will have to be absolutely positioned so lets creating a wrapping view, and 2 pieces. We'll style each piece `leftHeart` and `rightHeat`. Then setup some styles.


```
var Heart = React.createClass({
    render: function() {
        return (
            <View {...this.props} style={[styles.heart, this.props.style]}>
                <View style={styles.leftHeart} />
                <View style={styles.rightHeart} />
            </View>
        )
    }
})

//Styles

  heart: {
    width: 50,
    height: 50
  },
  heartShape: {
    width: 30,
    height: 45,
    position: 'absolute',
    top: 0,
    borderTopLeftRadius: 15,
    borderTopRightRadius: 15,
    backgroundColor: '#6427d1',
  },
  leftHeart: {
    transform: [
        {rotate: '-45deg'}
    ],
    left: 5
  },
  rightHeart: {
    transform: [
        {rotate: '45deg'}
    ],
    right: 5
  }

```
We set a width and height on the wrapping heart so it takes up space. We move all default styling into `heartShape` style and give it a nice purple color. Then we just adjust each left and right heart. The `leftHeart` piece will be on the left side, and rotated `-45deg` aka `45deg` towards the left, and the right will be the reverse.

# Setup Base Rendering

```
var HeartFloater = React.createClass({
  getInitialState: function() {
    return {};
  },

  render: function() {
    return (
      <View style={styles.container}>
        <Animated.View style={styles.heartWrap}>
          <Heart />
        </Animated.View>
      </View>
    );
  }
});


var styles = StyleSheet.create({
  container: {
    flex: 1
  },
  heartWrap: {
      position: 'absolute',
      bottom: 50,
      right: (deviceWidth/2) - 25
  },
  heart: {
    width: 50,
    height: 50
  },
  heartShape: {
    width: 30,
    height: 45,
    position: 'absolute',
    top: 0,
    borderTopLeftRadius: 15,
    borderTopRightRadius: 15,
    backgroundColor: '#6427d1',
  },
  leftHeart: {
    transform: [
        {rotate: '-45deg'}
    ],
    left: 5
  },
  rightHeart: {
    transform: [
        {rotate: '45deg'}
    ],
    right: 5
  }
});
```

Nothing much to call out here, except we add a new `heartWrap` class. We'll leave the heart be a static heart and then just animate a wrapper. The `heartWrap` is positioned absolutely and centered.


# Move the Heart


```
var HeartFloater = React.createClass({
  getInitialState: function() {
    return {
      position: new Animated.Value(0)
    };
  },
  componentDidMount: function() {
    Animated.timing(this.state.position, {
      duration: 2000,
      toValue: NEGATIVE_END_Y
    }).start();

  },
  getHeartAnimationStyle: function() {
    return {
      transform: [
        {translateY: this.state.position},
      ]
    }
  },
  render: function() {
    return (
      <View style={styles.container}>
        <Animated.View style={[styles.heartWrap, this.getHeartAnimationStyle()]}>
          <Heart />
        </Animated.View>
      </View>
    );
  }
});
```

We only setup a basic `Animated.Value` instead of an `Animated.ValueXY` because we will interpolate all the necessary effects and even the `X` position from the `Y` value.

We kick off an animation when the component mounts to animate the heart from `0` to negative half `deviceHeight`. We do this since translateY moves up when it is negative, and moves down when it is positive. We do this animation for 2 seconds.

# Fade the Heart

Now this is where we are going to get a little tricky with interpolation.

```
  componentWillMount: function() {
    this._yAnimation = this.state.position.interpolate({
      inputRange: [NEGATIVE_END_Y, 0],
      outputRange: [ANIMATION_END_Y, 0]
    });

    this._opacityAnimation = this._yAnimation.interpolate({
      inputRange: [0, ANIMATION_END_Y],
      outputRange: [1, 0]
    })

  },

  getHeartAnimationStyle: function() {
    return {
      transform: [
        {translateY: this.state.position},
      ],
      opacity: this._opacityAnimation
    }
  },
```
You can interpolate on an interpolated value. So we'll map our negative values, directly to positive values and also flip the step scale.

Our animation would typically go from `0` to `-300` but our new `_yAnimation` will flip that and stay we are animation goes from `0` to `300`.

This allows our animations like `opacity` to make more sense. We can interpolate on our `this._yAnimation` and specify `[0, ANIMATION_END_Y]` is tied to the opacity values `[1,0]`. So `0` aka the start is at `opacity` of `1 ` and moves to `0` which is tied to the `ANIMATION_END_Y` (roughly 300 depending on your device).

Then we pass the interpolated opacity value in `opacity`.

# Scale the Heart

```
    this._scaleAnimation = this._yAnimation.interpolate({
      inputRange: [0, 15, 30],
      outputRange: [0, 1.2, 1],
      extrapolate: 'clamp'
    })

    getHeartAnimationStyle: function() {
    return {
      transform: [
        {translateY: this.state.position},
        {scale: this._scaleAnimation}
      ],
      opacity: this._opacityAnimation
    }
  },
```

Now we'll add another animation below the other that will handle the scale. Because our now interpolated value runs positively we can just define pixel scale steps. So `0` maps to `0` so on creation the heart doesn't exist. The heart will quickly scale from `0` to `1.2x` it's size over the first `15` pixels it travels, then from `15` to `30` pixels it'll scale back down from `1.2` to `1`. 

This gives us a quick little pulse of the heart. We must add the `extrapolate: clamp` otherwise the heart will start scaling down and eventually go negative causing the heart to flip.

# Rotate/Sway the Heart

Almost done! Now lets make it sway a little bit. This one is a little difficult to fine tune, but I errored on the less dramtic side of things.

```
this._xAnimation = this._yAnimation.interpolate({
  inputRange: [0, ANIMATION_END_Y/2, ANIMATION_END_Y],
  outputRange: [0, 15, 0]
})

this._rotateAnimation = this._yAnimation.interpolate({
  inputRange: [0, ANIMATION_END_Y/4, ANIMATION_END_Y/3, ANIMATION_END_Y/2, ANIMATION_END_Y],
  outputRange: ['0deg', '-2deg', '0deg', '2deg', '0deg']
});

getHeartAnimationStyle: function() {
  return {
    transform: [
      {translateY: this.state.position},
      {translateX: this._xAnimation},
      {scale: this._scaleAnimation},
      {rotate: this._rotateAnimation}
    ],
    opacity: this._opacityAnimation
  }
},
```

This causes one sway to happen for `15` pixels then come back to the center. There are three rotations that happen that will make it look like it's wobbling a bit more.

The `_xAnimation` input range is from `0` to half the animation, to the end. Since our value of the `_xAnimation` interpolation is the end value of the animation we don't need to `extrapolate: clamp` here.

The `_rotateAnimation` takes `5` different steps. We just divide by each step. We start at `0`, then divide by 4 for quarter of the height animation, then a third of animation, half of the animation, then finally rotate back to 0 to finish the animation off. I pulled these out of thin air, and it looks okay but could use some fine tuning.

# Make AnimatedHeart component

Now lets make it show a bunch of hearts when we press down. Each press should put another heart onto an array, and when the heart is done animating we should remove it.

First move all the code to an `AnimatedHeart` like so.

```
var AnimatedHeart = React.createClass({
  getDefaultProps: function() {
    return {
      onComplete: function() {}
    };
  },
  getInitialState: function() {
    return {
      position: new Animated.Value(0)
    };
  },
  componentWillMount: function() {
    this._yAnimation = this.state.position.interpolate({
      inputRange: [NEGATIVE_END_Y, 0],
      outputRange: [ANIMATION_END_Y, 0]
    });

    this._opacityAnimation = this._yAnimation.interpolate({
      inputRange: [0, ANIMATION_END_Y],
      outputRange: [1, 0]
    });

    this._scaleAnimation = this._yAnimation.interpolate({
      inputRange: [0, 15, 30],
      outputRange: [0, 1.2, 1],
      extrapolate: 'clamp'
    });

    this._xAnimation = this._yAnimation.interpolate({
      inputRange: [0, ANIMATION_END_Y/2, ANIMATION_END_Y],
      outputRange: [0, 15, 0]
    })

    this._rotateAnimation = this._yAnimation.interpolate({
      inputRange: [0, ANIMATION_END_Y/4, ANIMATION_END_Y/3, ANIMATION_END_Y/2, ANIMATION_END_Y],
      outputRange: ['0deg', '-2deg', '0deg', '2deg', '0deg']
    });
  },
  componentDidMount: function() {
    Animated.timing(this.state.position, {
      duration: 2000,
      toValue: NEGATIVE_END_Y
    }).start(this.props.onComplete);
  },
  getHeartAnimationStyle: function() {
    return {
      transform: [
        {translateY: this.state.position},
        {translateX: this._xAnimation},
        {scale: this._scaleAnimation},
        {rotate: this._rotateAnimation}
      ],
      opacity: this._opacityAnimation
    }
  },
  render: function() {
    return (
        <Animated.View style={[styles.heartWrap, this.getHeartAnimationStyle(), this.props.style]}>
          <Heart />
        </Animated.View>
    )
  }
})
```

We modified a few things. We adjusted our `style` to include `this.props.style`.
Also we added a callback when the animation is finished.

```
  componentDidMount: function() {
    Animated.timing(this.state.position, {
      duration: 2000,
      toValue: NEGATIVE_END_Y
    }).start(this.props.onComplete);
  },
```

# Add AnimatedHeart on press

```
//https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random
function getRandomNumber(min, max) {
  return Math.random() * (max - min) + min;
}

var HeartFloater = React.createClass({
  getInitialState: function() {
    return {
      hearts: [] 
    };
  },
  addHeart: function() {
    startCount += 1;
    this.state.hearts.push({
      id: startCount,
      right: getRandomNumber(50, 150)
    });
    this.setState(this.state);
  },
  removeHeart: function(v) {
    var index = this.state.hearts.findIndex(function(heart) { return heart.id === v});
    this.state.hearts.splice(index, 1);
    this.setState(this.state);
  },
  render: function() {
    return (
      <View style={styles.container}>
        <TouchableWithoutFeedback style={styles.container} onPress={this.addHeart}>
          <View style={styles.container}>
            {
              this.state.hearts.map(function(v, i) {
                return (
                    <AnimatedHeart 
                      key={v.id}
                      onComplete={this.removeHeart.bind(this, v.id)}
                      style={{right: this.state.hearts[i].right}}
                    />
                ) 
              }, this)
            }
          </View>
        </TouchableWithoutFeedback>
      </View>
    );
  }
});
```

Our HeartFloater, now has a hearts array to hold our hearts for state. We've adjusted our `render` to use `TouchableWithoutFeedback` to call `addHeart` on press. 

Each press will increase our `startCount` which we use as an id generator. Also we add a random `right` style. a

Our render just loops over each heart, and renders the `AnimatedHeart` with a `key` which is essential for performance, an `onComplete` which is called when our animation is finished, and the right style we randomly generated.

On complete we call remove. Which finds the index of the heart, splices it, and sets state.

# Fix Background Colors and Styles

```
  heartWrap: {
      position: 'absolute',
      bottom: 30,
      backgroundColor: 'transparent'
  },
  heart: {
    width: 50,
    height: 50,
    backgroundColor: 'transparent'
  },
```

Without setting the background to transparent the background will be white causing one heart to clip another. This will prevent that.

We also removed the `right` position since we generate random positions between `50` and `150`.


# Final 

Thank you to Anthony Webb for the submission. As always check out the live code on RNPlay.

### Live code: [https://rnplay.org/apps/8VhSjw](https://rnplay.org/apps/8VhSjw)

If you any other inquiries do let me know and I'll show you how to build them.

{% img http://i.imgur.com/5JhgzQV.gif Purple floating fading swaying rotating hearts %}

# Final Code

```
var React = require('react-native');
var Dimensions = require('Dimensions');

var {
  width: deviceWidth,
  height: deviceHeight
} = Dimensions.get('window');

var {
  AppRegistry,
  StyleSheet,
  View,
  Animated,
  TouchableWithoutFeedback
} = React;

var ANIMATION_END_Y = Math.ceil(deviceHeight * .5);
var NEGATIVE_END_Y = ANIMATION_END_Y * -1;
var startCount = 1;

function getRandomNumber(min, max) {
  return Math.random() * (max - min) + min;
}

var Heart = React.createClass({
    render: function() {
        return (
            <View {...this.props} style={[styles.heart, this.props.style]}>
                <View style={[styles.leftHeart, styles.heartShape]} />
                <View style={[styles.rightHeart, styles.heartShape]} />
            </View>
        )
    }
});

var AnimatedHeart = React.createClass({
  getDefaultProps: function() {
    return {
      onComplete: function() {}
    };
  },
  getInitialState: function() {
    return {
      position: new Animated.Value(0)
    };
  },
  componentWillMount: function() {
    this._yAnimation = this.state.position.interpolate({
      inputRange: [NEGATIVE_END_Y, 0],
      outputRange: [ANIMATION_END_Y, 0]
    });

    this._opacityAnimation = this._yAnimation.interpolate({
      inputRange: [0, ANIMATION_END_Y],
      outputRange: [1, 0]
    });

    this._scaleAnimation = this._yAnimation.interpolate({
      inputRange: [0, 15, 30],
      outputRange: [0, 1.2, 1],
      extrapolate: 'clamp'
    });

    this._xAnimation = this._yAnimation.interpolate({
      inputRange: [0, ANIMATION_END_Y/2, ANIMATION_END_Y],
      outputRange: [0, 15, 0]
    })

    this._rotateAnimation = this._yAnimation.interpolate({
      inputRange: [0, ANIMATION_END_Y/4, ANIMATION_END_Y/3, ANIMATION_END_Y/2, ANIMATION_END_Y],
      outputRange: ['0deg', '-2deg', '0deg', '2deg', '0deg']
    });
  },
  componentDidMount: function() {
    Animated.timing(this.state.position, {
      duration: 2000,
      toValue: NEGATIVE_END_Y
    }).start(this.props.onComplete);
  },
  getHeartAnimationStyle: function() {
    return {
      transform: [
        {translateY: this.state.position},
        {translateX: this._xAnimation},
        {scale: this._scaleAnimation},
        {rotate: this._rotateAnimation}
      ],
      opacity: this._opacityAnimation
    }
  },
  render: function() {
    return (
        <Animated.View style={[styles.heartWrap, this.getHeartAnimationStyle(), this.props.style]}>
          <Heart />
        </Animated.View>
    )
  }
})

var HeartFloater = React.createClass({
  getInitialState: function() {
    return {
      hearts: [] 
    };
  },
  addHeart: function() {
    startCount += 1;
    this.state.hearts.push({
      id: startCount,
      right: getRandomNumber(50, 150)
    });
    this.setState(this.state);
  },
  removeHeart: function(v) {
    var index = this.state.hearts.findIndex(function(heart) { return heart.id === v});
    this.state.hearts.splice(index, 1);
    this.setState(this.state);
  },
  render: function() {
    return (
      <View style={styles.container}>
        <TouchableWithoutFeedback style={styles.container} onPress={this.addHeart}>
          <View style={styles.container}>
            {
              this.state.hearts.map(function(v, i) {
                return (
                    <AnimatedHeart 
                      key={v.id}
                      onComplete={this.removeHeart.bind(this, v.id)}
                      style={{right: this.state.hearts[i].right}}
                    />
                ) 
              }, this)
            }
          </View>
        </TouchableWithoutFeedback>
      </View>
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1
  },
  heartWrap: {
      position: 'absolute',
      bottom: 30,
      backgroundColor: 'transparent'
  },
  heart: {
    width: 50,
    height: 50,
    backgroundColor: 'transparent'
  },
  heartShape: {
    width: 30,
    height: 45,
    position: 'absolute',
    top: 0,
    borderTopLeftRadius: 15,
    borderTopRightRadius: 15,
    backgroundColor: '#6427d1',
  },
  leftHeart: {
    transform: [
        {rotate: '-45deg'}
    ],
    left: 5
  },
  rightHeart: {
    transform: [
        {rotate: '45deg'}
    ],
    right: 5
  }
});

AppRegistry.registerComponent('animate_slide', () => HeartFloater);

```
