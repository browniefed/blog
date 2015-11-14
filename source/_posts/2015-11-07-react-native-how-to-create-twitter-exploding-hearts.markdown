---
layout: post
title: "React Native - How to create Twitter exploding hearts"
date: 2015-11-07 08:55
comments: true
categories: react react-native twitter animations react-art heart exploding
facebook:
    image: http://i.imgur.com/4OrRx7Y.png
twitter_card:
    creator: browniefed
    type: summary_large_image
    image: http://i.imgur.com/4OrRx7Y.png
---

So we're going to build this exploding heart, except just know Twitter kind of cheated. Not really but they used an image and played each frame adjusting `background-position` so it looked animated. Okay not cheated they used a really smart technique but what's the fun in doing that when we can build it for real!

I've already built a Firework concept here [http://browniefed.com/blog/2015/08/29/react-native-animated-with-react-art-firework-show/](http://browniefed.com/blog/2015/08/29/react-native-animated-with-react-art-firework-show/) and about Hearts here [http://browniefed.com/blog/2015/09/07/react-native-periscope-hearts-animation/](http://browniefed.com/blog/2015/09/07/react-native-periscope-hearts-animation/) now we just need to bring them together.

For the record this is what theirs looks like here [http://i.imgur.com/3a8PiSY.png](http://i.imgur.com/3a8PiSY.png)


## What are we building


{% img http://i.imgur.com/lMOxqIL.gif So not lazy like Twitter %}

<!-- more -->

## Concept

Rather than iterating over 28 separate image frames what we're going to do instead is make some pseudo-keyframe-animations. We'll use `Animated` of course. We'll define our range from `0` to `28`. Meaning we'll have 28 frames to deal with. 

This makes the math easy, because we can walk through each frame, and describe what the animation should look like for each frame.

We could create separate animated values for each property and coordinate the animation with `Animated.sequence` and `Animated.parallel` but I prefer interpolation. YMMV


## Setup

```
var React = require('react-native');

var {
  AppRegistry,
  StyleSheet,
  View,
  Dimensions,
  TouchableWithoutFeedback,
  Animated
} = React;

var Art = require('ReactNativeART');

var {
  Surface,
  Group,
  Shape,
  Path
} = Art;

var AnimatedShape = Animated.createAnimatedComponent(Shape);

var {
  width: deviceWidth,
  height: deviceHeight
} = Dimensions.get('window');

```
We'll need to setup `ReactNativeArt` in XCode, you can check out how to do that [here](http://browniefed.com/blog/2015/05/03/getting-react-art-running-on-react-native/).

Things to call out here is we are creating a custom animated view, called `AnimatedShape`. We pass in the `art` `Shape` component into `Animated.createAnimatedComponent`. This allows us to use `Animated` values in any component.

## More Setup

```
var HEART_SVG = "M130.4-0.8c25.4 0 46 20.6 46 46.1 0 13.1-5.5 24.9-14.2 33.3L88 153.6 12.5 77.3c-7.9-8.3-12.8-19.6-12.8-31.9 0-25.5 20.6-46.1 46-46.2 19.1 0 35.5 11.7 42.4 28.4C94.9 11 111.3-0.8 130.4-0.8"
var HEART_COLOR = 'rgb(226,38,77,1)';
var GRAY_HEART_COLOR = "rgb(204,204,204,1)";

var FILL_COLORS = [
  'rgba(221,70,136,1)',
  'rgba(212,106,191,1)',
  'rgba(204,142,245,1)',
  'rgba(204,142,245,1)',
  'rgba(204,142,245,1)',
  'rgba(0,0,0,0)'
];

var PARTICLE_COLORS = [
  'rgb(158, 202, 250)',
  'rgb(161, 235, 206)',
  'rgb(208, 148, 246)',
  'rgb(244, 141, 166)',
  'rgb(234, 171, 104)',
  'rgb(170, 163, 186)'
]
```

More setup here. We create our heart SVG path to render, and setup a bunch of colors that we will use in our animations later. We need to set stuff up as `rgb` or `rgba` so that `Animated` can interpolate it correctly as at the moment it cannot do hex values.

```

function getXYParticle(total, i, radius) {
  var angle = ( (2*Math.PI) / total ) * i;

  var x = Math.round((radius*2) * Math.cos(angle - (Math.PI/2)));
  var y = Math.round((radius*2) * Math.sin(angle - (Math.PI/2)));
  return {
    x: x,
    y: y
  }
}

function getRandomInt(min, max) {
  return Math.floor(Math.random() * (max - min)) + min;
}

function shuffleArray(array) {
    for (var i = array.length - 1; i > 0; i--) {
        var j = Math.floor(Math.random() * (i + 1));
        var temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }
    return array;
}

```
Yes more setup. `getXYParticle` is a function I've modified from the fireworks particle blog post I wrote. This essentially distributes a random number of particles around a circle. This is what we will use for the little particles that blow up.

The `getRandomInt` is pretty self explanatory, but it just returns a random number and we'll use it to create some variance.

`shuffleArray` also just shuffles things in an array, we'll use this to achieve random color effects later.


## Blank Canvas

```
var ExplodingHearts = React.createClass({
  getInitialState: function() {
    return {
      animation: new Animated.Value(0) 
    };
  },
  explode: function() {
    Animated.timing(this.state.animation, {
      duration: 1500,
      toValue: 28
    }).start(() => {
      this.state.animation.setValue(0);
      this.forceUpdate();
    }.bind(this));
  },

  render: function() {
    return (
      <View style={styles.container}>
        <TouchableWithoutFeedback onPress={this.explode} style={styles.container}> 
          <View>
            <Surface width={deviceWidth} height={deviceHeight}>
            </Surface>
          </View>
        </TouchableWithoutFeedback>
      </View>
    );
  }
});

```

We setup a blank canvas. We need to create our initial `animation` value so we just do a `new Animated.Value(0)` in our `getInitialState`.

We setup a `TouchableWithoutFeedback` for the ability to use `onPress` to trigger our animation. We then setup our `ART` `Surface` to fill the screen.

Why are we using ART for this? Well it'll make our rendering and animations very efficient, we could use a bunch of `Views` for this demo but on a large scale with lots of graphics work you should use `ART` 

Our `explode` function kicks off our animation and we do it over 1.5 seconds. We will animate to 28 because that is how many "frames" we have.

Don't worry about the callback, that's just to reset the animation when it is done, and also the `forceUpdate` re-renders so we get new random values on each subsequent trigger.

## Render A Heart

```
<Surface width={deviceWidth} height={deviceHeight}>
    <AnimatedShape
      d={HEART_SVG}
      x={0}
      y={0}
      scale={1}
      fill={GRAY_HEART_COLOR}
    />
</Surface>
```

We just render a heart, pass in our SVG path, put it at the top left `0,0` coordinates. Scale is set to 1 and we fill it with our `GRAY_HEART_COLOR` we setup above.


## Talking Keyframes and Animations

Before we dive in I want to explain what is about to happen. The original Twitter exploding has 28 frames. Our animation will start at `0`, and that is our default state. So we'll need to set everything up to default when we start `0` (initial render). 

Then anytime from `1` to `28` we will need to design our values so that they produce the correct frame animation.

`Animated` provides a way to interpolate. What that means is given a value, we want it to go through a formula and spit out another value. `Animated` does this via ranges, which can at times be a little inflexible and we have to hack around it's shortcomings to get desired effects.

All (well almost all) our animations will be interpolating from the single `this.state.animation` that we created earlier. This just makes it easy to comprehend and layout your animation frames. Because you can then specify that something happens at frame `10` instead of dividing `1/28` and say start at `0.03571428571`.



## Scale it up

```
render: function() {
    var heart_scale = this.state.animation.interpolate({
      inputRange: [0, .01, 6, 10, 12, 18, 28],
      outputRange: [1, 0, .1, 1, 1.2, 1, 1],
      extrapolate: 'clamp'
    });

    var heart_fill = this.state.animation.interpolate({
      inputRange: [0, 2],
      outputRange: [GRAY_HEART_COLOR, HEART_COLOR],
      extrapolate: 'clamp'
    })

    var heart_x = heart_scale.interpolate({
      inputRange: [0, 1],
      outputRange: [90, 0],
    })

    var heart_y = heart_scale.interpolate({
      inputRange: [0, 1],
      outputRange: [75, 0],
    })
}
```

Alright there is a lot going on. Lets break it down.

```
    var heart_scale = this.state.animation.interpolate({
      inputRange: [0, .01, 6, 10, 12, 18],
      outputRange: [1, 0, .1, 1, 1.2, 1],
      extrapolate: 'clamp'
    });
```
We `interpolate` on the `this.state.animation` and give it an `inputRange` and `outputRange` array. These must have the same amount of array values. 

I talked about some weird things with `Animated` and setting up defaults. Well `inputRange: [0, .01], outputRange: [1, 0]` is the first example. 

At 0 we want it to be fully scaled, so output at 1. However as soon as the animation is triggered we want it to be at 0. If we only specified `0, 1` as the inputRange, it would have the heart scale down from 1 to 0. So specifying the scale inputRange at `0 => .01` means it'll basically disappear.

It's essentially a way to make an animation not a whole frame, and or happen virtually immediately.

There is a slight spring in the heart. So from frame `10` to `12` it will spring up fast to `1.2` scale, and then slowly fall back from `1.2` to `1` from frames `12` to `18`.

`extrapolate: 'clamp'` IS EXTREMELY IMPORTANT HERE. If we want it to just stay the same once it hits frame 18 and not do anything else until the end we must add the clamp. Otherwise it will continue to animate at the current stepping value, so it would scale down below 1 and we don't want that.

```
    var heart_fill = this.state.animation.interpolate({
      inputRange: [0, 2],
      outputRange: [GRAY_HEART_COLOR, HEART_COLOR],
      extrapolate: 'clamp'
    })
```
We have the heart hidden after frame `1`, so what this animation says is at `0` , inital render, be gray. Anytime from frame 2 and out be red, and always be read.

```

    var heart_x = heart_scale.interpolate({
      inputRange: [0, 1],
      outputRange: [90, 0],
    })

    var heart_y = heart_scale.interpolate({
      inputRange: [0, 1],
      outputRange: [75, 0],
    })
```

This is the one part where we don't want to base things on the key frame. Because there is no `transform-origin` like there is in CSS, the default scale will scale out to the top left.
That isn't what we want. 

So to scale out to the center we need to animate the x/y while scaling, so we interpolate off the interpolate for scale. Remember we default our scale up above to 1, so we are reversing stuff here saying when the scale is scaling down from 0 to 1 adjust to 90 for x and 75 for y.

The 90/75 just has to deal with the current surface center.

```
    return (
        <Surface width={deviceWidth} height={deviceHeight}>
            <AnimatedShape
              d={HEART_SVG}
              x={heart_x}
              y={heart_y}
              scale={heart_scale}
              fill={heart_fill}
            />
        </Surface>

    )
```

We use our `AnimatedShape` and pass in the animated values we created. A lot of things I explain up above are the basic concepts through out this tutorial so I wont' explain them again.

## Add a Circle

```
var AnimatedCircle = React.createClass({displayName: "Circle",
  render: function() {
    var radius = this.props.radius;
    var path = Path().moveTo(0, -radius)
        .arc(0, radius * 2, radius)
        .arc(0, radius * -2, radius)
        .close();
    return React.createElement(AnimatedShape, React.__spread({},  this.props, {d: path}));
  }
});
```

This is taken from previous firework demos and default ReactART, but I've converted it to use our `AnimatedShape` we created up above.

```
<Surface width={deviceWidth} height={deviceHeight}>
  <Group x={100} y={200}>
    <AnimatedShape
      d={HEART_SVG}
      x={heart_x}
      y={heart_y}
      scale={heart_scale}
      fill={heart_fill}
    />
    <AnimatedCircle
      x={90}
      y={75}
      radius={150}
      scale={1}
      strokeWidth={3}
      fill="#000"
      opacity={1}
    />
  </Group>
</Surface>
```

We render an arbitrary circle, at `90` and `75` which is the center of our current surface.

## Blow that Circle Up

```
    var circle_scale = this.state.animation.interpolate({
      inputRange: [0, 1, 4],
      outputRange: [0, .3, 1],
      extrapolate: 'clamp'
    });

    var circle_stroke_width = this.state.animation.interpolate({
      inputRange: [0, 5.99, 6, 7, 10],
      outputRange: [0, 0, 15, 8, 0],
      extrapolate: 'clamp'
    });

    var circle_fill_colors = this.state.animation.interpolate({
      inputRange: [1, 2, 3, 4, 4.99, 5],
      outputRange: FILL_COLORS,
      extrapolate: 'clamp'
    })

    var circle_opacity = this.state.animation.interpolate({
      inputRange: [1,9.99, 10],
      outputRange: [1, 1, 0],
      extrapolate: 'clamp'
    })
```

Alright so we scale up, based on the image we scale up to `.3` in a single frame, then over the course of 3 frames we scale up our circle to a scale of 1.

Our stroke width also changes however we won't always render it. We only need it for a few frames starting at frame 6. So we will specify that the stroke width stays at `0` from `0` to frame `5.99`.

Then over the course of 1 frame it goes to 15 which I chose at random, down to 8, and eventually 0 by frame 10.

We specify a range of colors in the array I talked about in setup. The weird part is that we have to specify the final color a few times before setting the fill to transparent. The reason is that if we specify just the transparent color the purple color will fade out to transparent but we want it to completely disappear and just show the stroke.

So we have to coordinate stroke frames appearing with the fill color disappearing.

Finally our opacity stays 1 until frame 9.99 (the end of our circle stroke) then we kill it on frame 10.


```
    <AnimatedCircle
      x={89}
      y={75}
      radius={150}
      scale={circle_scale}
      strokeWidth={circle_stroke_width}
      stroke={FILL_COLORS[2]}
      fill={circle_fill_colors}
      opacity={circle_opacity}
    />
```
We put those in and because our stroke color is always the same we just reference our fill colors.

## Bunch of Circles Blowing Up
```
<Surface width={deviceWidth} height={deviceHeight}>
  <Group x={100} y={200}>
    <AnimatedShape
      d={HEART_SVG}
      x={heart_x}
      y={heart_y}
      scale={heart_scale}
      fill={heart_fill}
    />
    <AnimatedCircle
      x={90}
      y={75}
      radius={150}
      scale={circle_scale}
      strokeWidth={circle_stroke_width}
      stroke={FILL_COLORS[2]}
      fill={circle_fill_colors}
      opacity={circle_opacity}
    />

    {this.getSmallExplosions(150, {x:90, y:75})}
  </Group>
</Surface>
```

Alright now we setup our final piece. Calling `getSmallExplosions` with some data. In our case the radius of our circle and the central coordinates.

```
  getSmallExplosions: function(radius, offset) {
    return [0,1,2,3,4,5,6].map((v, i, t) => {

      var scaleOut = this.state.animation.interpolate({
        inputRange: [0, 5.99, 6, 13.99, 14, 21],
        outputRange: [0, 0, 1, 1, 1, 0],
        extrapolate: 'clamp'
      });

      var moveUp = this.state.animation.interpolate({
        inputRange: [0, 5.99, 14],
        outputRange: [0, 0, -15],
        extrapolate: 'clamp'
      });

      var moveDown = this.state.animation.interpolate({
        inputRange: [0, 5.99, 14],
        outputRange: [0, 0, 15],
        extrapolate: 'clamp'
      });

      var color_top_particle = this.state.animation.interpolate({
        inputRange: [6, 8, 10, 12, 17, 21],
        outputRange: shuffleArray(PARTICLE_COLORS)
      })

      var color_bottom_particle = this.state.animation.interpolate({
        inputRange: [6, 8, 10, 12, 17, 21],
        outputRange: shuffleArray(PARTICLE_COLORS)
      })

      var position = getXYParticle(7, i, radius)

      return (
        <Group 
          x={position.x + offset.x } 
          y={position.y + offset.y} 
          rotation={getRandomInt(0, 40) * i}
        >
          <AnimatedCircle 
            x={moveUp}
            y={moveUp}
            radius={15} 
            scale={scaleOut} 
            fill={color_top_particle} 
          />
          <AnimatedCircle 
            x={moveDown}
            y={moveDown}
            radius={8} 
            scale={scaleOut} 
            fill={color_bottom_particle} 
          />
        </Group>
      )
    }, this)
  },
```

Not going to explain this one too indepth or I'll keep repeating myself. We create a bunch of animations for each particle but add some randomness into the mix. We call `shuffleArray` on our `PARTICLE_COLORS` so over the course of the frames it is active it'll animate to random colors for each particle.

Also we add a bit of `rotation` to the group, so when we animate the particles up/down they'll go in all different directions.

## Done

Now you too can add a new interaction that all of your users will despise! No this isn't a perfect replica of the the Twiter animation because I added color variance and some random rotation to the small exploding/shrinking circles.

{% img http://i.imgur.com/lMOxqIL.gif That is hot %}


### Interactive Demo at [https://rnplay.org/apps/nJjHdw](https://rnplay.org/apps/nJjHdw)


I'm not posting the full code, this is a long one. Just check it out on RNPlay.