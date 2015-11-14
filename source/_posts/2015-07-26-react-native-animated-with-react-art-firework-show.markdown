---
layout: post
title: "React-Native Animated with React-Art - Firework Tap To Shoot"
date: 2015-08-29 22:12
comments: true
categories: react-native react react-art animated firework shooter tap animations
---

#Introduction
This actually could be built with out react-art, but we'll use react-art just for good measure.
What are we building? A firework show. Nothing fancy. Just tap on the screen and a firework will be shot to that point and explode.

#What?

This is what we're building

{% img http://i.imgur.com/Dj60a6e.gif Firework Shooter %}


<!-- more -->

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

Lets think about this. 
We want a mortar (a small glowing ball) to shoot from the bottom center of the screen to where we've tapped.

That means with `react-art` we'll have to create a closed `Path` that is a circle. It just so happens that `react-art` ships with a `Circle` shape for us to use.

Okay so our list of needs

* A press handler to get where we tapped
* A `Circle` that can take Animated API props
* A way to animate that `Circle` aka mortar to the tap
* Render a mortar on the `Surface`

```
<TouchableWithoutFeedback onPress={this._handleAddFirework}>
```
We'll use the `TouchableWithoutFeedback` to call out to a function to queue up adding a firework.
Check that off the list.

```
//React-Art ships with this component however not react-native-art implementation we'll just grab it and
//modify this to use the AnimatedShape we create up above. Thanks Facebook :)

/**
 * Copyright 2013-2014 Facebook, Inc.
 * All rights reserved.
 *
 * This source code is licensed under the BSD-style license found in the
 * LICENSE file in the root directory of this source tree. An additional grant
 * of patent rights can be found in the PATENTS file in the same directory.
 *
 * @providesModule Circle.art
 * @typechecks
 *
 * Example usage:
 * <Circle
 *   radius={10}
 *   stroke="green"
 *   strokeWidth={3}
 *   fill="blue"
 * />
 *
 */
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

Hey look we just grabbed it and changed `Shape` to `AnimatedShape`. Yaye for React and reusable code.

```
var MORTAR_RADIUS = 5;
///...
_handleAddFirework: function(e) {
    var _shootingPosition = new Animated.ValueXY({x: width/2, y: height - MORTAR_RADIUS});

    this.state.fireworks.push({
      shootingPosition: _shootingPosition,
    });

    Animated.timing(_shootingPosition, {
        duration: 300,
        toValue: {
          y: e.nativeEvent.locationY,
          x: e.nativeEvent.locationX
        }
    }).start()
    
    this.setState(this.state);
}
```
Alright. Pause and lets analyze this code.

First line `_shootingPosition`, we create a new `Animated.ValueXY` and set our defaults. These defaults are the starting position of the animation. Usually these would default to `x: 0, y:0` but we have other plans.

We set our `x` to `width/2` which is the middle of the phone. Then our `y` is going to the `height` of the device minus a `MORTAR_RADIUS`. We put a constant at the top of the code to say our mortar radius is going to be 5.

Next we add it to an array of fireworks we'll shoot later.

Then we create the animation.
We want the mortar to take `300` milliseconds to reach the spot the user pressed. We set the `toValue` to where our users pressed.
Then we start the animation. Yeah we haven't even rendered anything yet but it'll all be okay trust me.

Finally we set our state and thus it'll cause a re-render and we can render our firework.


```
<Surface width={width} height={height}>
{
    this.state.fireworks.map((firework) => {
        return <AnimatedCircle 
                    radius={MORTAR_RADIUS} 
                    x={firework.shootingPosition.x} 
                    y={firework.shootingPosition.y}
                    fill="#000"
                />
    })
}
</Surface>
```
So we map over each firework, return the `AnimatedCircle` and set the appropriate properties, which 2 of those `x`, and `y` are animated properties. In our case we're going to fill the cirlcle with black to start.

So what does that all look like? Something like this

{% img http://i.imgur.com/arIjyoz.gif Black Shooting Circles %}

# Shoot multiple mortars that disappear

Well mortars don't stick like that. So lets make them disappear. 
The `start()` function takes a callback that is called when the animation completes.

To identify the firework in the array we'll just use the `shootingPosition` animation to identify it and filter it out.

Something like this. 
    
```
  removeSelf: function(_shootingPosition) {
    this.state.fireworks = this.state.fireworks.filter((firework) => firework.shootingPosition !== _shootingPosition);
    this.setState(this.state);
  },

  ///
    Animated.timing(_shootingPosition, {
        duration: 300,
        toValue: {
          y: e.nativeEvent.locationY,
          x: e.nativeEvent.locationX
        }
    ]).start(this.removeSelf.bind(this, _shootingPosition));
```

Not the most elegant of solutions but hey it works. It runs through each firework, removes our _shootingPosition, and then refreshes the UI.

# Animate the color

Lets make the mortar more than just a black dot. Lets pretend it's a fire ball, and we want it to alternate between yellow and orange.

We'll use these 2 colors and pop them at the top

```
var SHOOTING_COLORS = [
  'rgb(234,238,112)', //Yellow
  'rgb(245,137,12)' //Orange
];
```

The `interpolate` function we're going to call only works with `rgb` hence the use of `rgb` instead of hex.

Next we'll need to create another Animated value.

Our code will look like so 

```
    var _shootingPosition = new Animated.ValueXY({x: width/2, y: height - MORTAR_RADIUS});
    var _shootingColor = new Animated.Value(0);

    this.state.fireworks.push({
      shootingPosition: _shootingPosition,
      shootingColor: _shootingColor,
    });

    //

    _shootingPosition.addListener(this.adjustShootingFill.bind(null, _shootingColor));
  
    adjustShootingFill: function(_shootingColor, value) {
        Animated.timing(_shootingColor, {
          duration: 16,
          toValue: _shootingColor.__getAnimatedValue() == 0 ? 1 : 0
        }).start()
    },

```
We start off by adding a new `Animated.Value`, and set it to 0. We add it to our firework object.

Then we add a listener to it. What `addListener` does is provides a callback that will be called each time the mortar position is updated.
The bind is just so it'll pass in our `_shootingColor` Animated value as the first argument.

We'll use the `Animated.timing` function again to transition it between colors over 16ms. We call `__getAnimatedValue()` and do the inverse of it.

So every `16ms` the mortar will transition from yellow => orange => yellow => orange, etc.


Now what does that look like in our render?

```
<Surface width={width} height={height}>
{
    this.state.fireworks.map((firework) => {
        
        var _shootingFill = firework.shootingColor.interpolate({
          inputRange: [0,1],
          outputRange: SHOOTING_COLORS
        });


        return <AnimatedCircle 
                    radius={MORTAR_RADIUS} 
                    x={firework.shootingPosition.x} 
                    y={firework.shootingPosition.y}
                    fill={_shootingFill}
                />
    })
}
</Surface>
```
We need to create an interpolator. This interpolate will encapsulate the logic that picks the color when we change the value of `shootingColor` up above in our `adjustShootingFill` function.

It maps to

```
0 => yellow
1 => orange
```
Then we pass that into our fill property and we're done.

{% img http://i.imgur.com/r69Ba6r.gif Orange Yellow Moratrs %}

# Make those mortars explode

Now for the fun part. Lets make our mortars explode.

Our mortar concept will be pretty simplistic, we'll create 20 circles that explode outwards and expand. 
We could create all sorts of tails that fly around and do cool things but that is a tutorial for another time.

First lets setup some variables

```
var PARTICLE_RADIUS = 30; // How big should the explosions be
var PARTICLE_COUNT = [1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20]; // How many particles per explosion, this is the lazy persons range call

var PARTICLE_COLORS = [
    'rgba(54, 17, 52, 100)',
    'rgba(176, 34, 140, 100)',
    'rgba(234, 55, 136, 100)',
    'rgba(229, 107, 112, 100)',
    'rgba(243, 145, 160, 100)'
]
```

We the `PARTICLE_RADIUS` which will determine how large each explosion is. We setup `PARTICLE_COUNT` which is the amount of particles we'll use.
Finally we want our firework show to be 5 different colors. So each explosion will change between each of these colors.

For the sake of simplicity we'll make each particle the same exact color meaning we only need 1 `Animated.Value` for it.

```
    var _particleColor = new Animated.Value(0);
    var _particleRadius = new Animated.Value(0);
    var _coreOpacity = new Animated.Value(1);

```
We also need an `Animated.Value` for our radius, we put it at 0 so it starts hidden. The other `Animated.Value` is our core opacity animation. 
That is the value for hiding our core mortar once it explodes.

```
    var _particlePositions = PARTICLE_COUNT.map(() => new Animated.ValueXY({x: 0, y: 0}));
```
Each particle will have to go to a different position so in this case we'll need 20 `Animated.ValueXY`.

```
this.state.fireworks.push({
      shootingPosition: _shootingPosition,
      shootingColor: _shootingColor,
      particleColor: _particleColor,
      particleRadius: _particleRadius,
      coreOpacity: _coreOpacity,
      particlePositions: _particlePositions
    });
```

Add them to our firework so we can access them later in our render.

```
    var _animatedParticles = [
        Animated.timing(_particleRadius, {
          duration: 700,
          toValue: 1
        }),
        Animated.timing(_coreOpacity, {
          duration: 200,
          toValue: 0
        })
    ]

    _movingParticles = _particlePositions.map((particle, i) => {
      var _xy = getXYParticle(PARTICLE_COUNT.length, i, PARTICLE_RADIUS);
      return Animated.timing(particle, {
        duration: 250,
        toValue: _xy
      })
    })

    //At the bottom of the file
    function getXYParticle(total, i, radius) {
      var angle = 360/total*i;

      var x = Math.round((radius*2) * Math.cos(angle - (Math.PI/2)));
      var y = Math.round((radius*2) * Math.sin(angle - (Math.PI/2)));

      return {
        x: x,
        y: y
      }
    }

    _animatedParticles = _animatedParticles.concat(_movingParticles);
```

This whole animation mapping is setting us up for what we are going to do next. Which is queue up our animations.

We create an array of animations that need to happen. The first is expanding each `_particleRadius`. We have determine that it will take 700 milliseconds to fully expand the explosion. The `_particleRadius` will actually be the particle scale. We'll just scale up the circle so it looks like it's exploding outwards, but that will be shown off in our render function.

We set our `_coreOpacity` aka our mortar ball to fade out and disappear over 200 milliseconds.

We need to create the animations for each particle. It needs to shoot out from our current mortar location to different points on the circle.
After googling around I found a function below, and deleted a bunch of stuff to basically get down to a rough position algorithim to return `x,y` positions.

We once again use `Animated.timing` to say that the particle should take 250 milliseconds to get into it's position.

```
    Animated.sequence([
      Animated.timing(_shootingPosition, {
        duration: 300,
        toValue: {
          y: e.nativeEvent.locationY,
          x: e.nativeEvent.locationX
        }
      }),
      Animated.parallel(_animatedParticles)
    ]).start(this.removeSelf.bind(this, _shootingPosition));
```

Now to queue up our animations. We want our mortar shooting to happen first, then the explosion.
We do that using `Animated.sequence`. 

We first have our mortar go up to the location of the users touch.
The next piece is wrapping all of our explosion animations in an `Animated.parallel`. This is the opposite of sequence, which it says execute all of these animations at the same time. 

So our mortar fading out and disappearing, our particles expanding, changing color, and exploding outward will all happen at the same time.


```
    _shootingPosition.addListener(this.adjustShootingFill.bind(null, _shootingColor));
    _particleRadius.addListener(this.adjustParticleFill.bind(null, _particleColor));

    ///
      adjustParticleFill: function(_particleColor, value) {
        var _currentFill = _particleColor.__getAnimatedValue(),
            _particleFill = _currentFill === 5 ? 0 : _currentFill + 1;

        Animated.timing(_particleColor, {
          duration: 16,
          toValue: _particleFill
        }).start()
      },
```

Finally we need to change our color of our particle. So like before we'll attach to the expanding of our particle and make it call a function to change the fill color.
In our new case we have five colors to choose from so the logic is a little different but mostly the same.


Now for our rendering of this all. We'll move it out to a different function to deal with.

```
  <Surface width={width} height={height}>
    {this.getFireworks()}
  </Surface>
```

```
getFireworks: function() {
    return this.state.fireworks.map((firework, i) => {

      var _shootingFill = firework.shootingColor.interpolate({
        inputRange: [0,1],
        outputRange: SHOOTING_COLORS
      });

      var _particleFill = firework.particleColor.interpolate({
        inputRange: [0,1,2,3,4],
        outputRange: PARTICLE_COLORS
      });

      return (
          <AnimatedGroup 
            x={firework.shootingPosition.x}
            y={firework.shootingPosition.y}
          >
                <AnimatedCircle
                  opacity={firework.coreOpacity}
                  radius={MORTAR_RADIUS}
                  fill={_shootingFill}
                />
                <Group>
                {
                  PARTICLE_COUNT.map((v, j) => {
                    return <AnimatedCircle
                      x={firework.particlePositions[j].x}
                      y={firework.particlePositions[j].y}
                      scaleX={firework.particleRadius}
                      scaleY={firework.particleRadius}
                      radius={PARTICLE_RADIUS}
                      fill={_particleFill}
                    />
                  })
                }
                </Group>
          </AnimatedGroup>
      );
    })
  },
```
There is a lot going on here. We have our same `_shootingFill` like we did before, that causes the yellow => orange interpolation.
We add another interpolation for the particleFill. Same concept as the `_shootingFill` just with 5 colors now.

What is much different here is the `AnimatedGroup`. We have moved our `x,y` from the `AnimatedCircle` to the `AnimatedGroup`.

This will bundle our mortar, and our particles all in the same coordinate area. That way we can have our particles start at `0,0` and move outwards and the `AnimatedGroup` will make sure all the coordinates are handled correctly.


```
<AnimatedCircle
  opacity={firework.coreOpacity}
  radius={MORTAR_RADIUS}
  fill={_shootingFill}
/>
```
You can see here we put out `coreOpacity` animation in there. So it'll fade from `1` opacity down to `0` over that 200 milliseconds we setup.

```
  <Group>
    {
      PARTICLE_COUNT.map((v, j) => {
        return <AnimatedCircle
          x={firework.particlePositions[j].x}
          y={firework.particlePositions[j].y}
          scaleX={firework.particleRadius}
          scaleY={firework.particleRadius}
          radius={PARTICLE_RADIUS}
          fill={_particleFill}
        />
      })
    }
  </Group>
```
We group the Particles with a `Group` component but that's just arbitrary.

We map over our 20 particles, and set the `x` and `y` to the points we had determined with our `getXYParticle` function.

You'll notice we are passing in `particleRadius` to the `scaleX` and `scaleY` properties.
This is because while writing this I realized that the `AnimatedCircle` takes the radius property and creates a path out of it. That is not animateable in this particular way, so the solution I came up was to scale each particle to 0. Basically making it completely hidden.

That then allows us to expand it out to it's full scale and make it look like an explosion. It actually works better.

Finally we add in our `PARTICLE_RADIUS` we defined at the top, aka size of each explosion, and put in our `_particleFill` which is the color interpolation between those 5 colors.


# Play with it

That is all! We have a firework shooter!

Check it out on RNPlay as per usual [https://rnplay.org/apps/ysm12A](https://rnplay.org/apps/ysm12A).

Tap to your hearts content and watch all the explosive animation goodness appear on screen.

{% img http://i.imgur.com/Dj60a6e.gif Firework Shooter %}


#Final Code

```
var React = require('react-native');
var Dimensions = require('Dimensions');
var ReactNativeART = require('ReactNativeART');

var {
  width,
  height
} = Dimensions.get('window');

var MORTAR_RADIUS = 5;
var PARTICLE_RADIUS = 30;
var PARTICLE_COUNT = [1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20];

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

var SHOOTING_COLORS = [
  'rgb(234,238,112)',
  'rgb(245,137,12)'
];

var PARTICLE_COLORS = [
    'rgba(54, 17, 52, 100)',
    'rgba(176, 34, 140, 100)',
    'rgba(234, 55, 136, 100)',
    'rgba(229, 107, 112, 100)',
    'rgba(243, 145, 160, 100)'
]

var AnimatedShape = Animated.createAnimatedComponent(Shape);
var AnimatedGroup = Animated.createAnimatedComponent(Group);

var FireworkShooter = React.createClass({
  getInitialState: function() {
    return {
      fireworks: []
    };
  },
  adjustShootingFill: function(_shootingColor, value) {
    Animated.timing(_shootingColor, {
      duration: 16,
      toValue: _shootingColor.__getAnimatedValue() == 0 ? 1 : 0
    }).start()
  },
  adjustParticleFill: function(_particleColor, value) {
    var _currentFill = _particleColor.__getAnimatedValue(),
        _particleFill = _currentFill === 5 ? 0 : _currentFill + 1;

    Animated.timing(_particleColor, {
      duration: 16,
      toValue: _particleFill
    }).start()
  },
  removeSelf: function(_shootingPosition) {
    this.state.fireworks = this.state.fireworks.filter((firework) => firework.shootingPosition !== _shootingPosition);
    this.setState(this.state);
  },
  _handleAddFirework: function(e) {
    var _shootingPosition = new Animated.ValueXY({x: width/2, y: height - MORTAR_RADIUS});
    var _shootingColor = new Animated.Value(0);

    var _particleColor = new Animated.Value(0);
    var _particleRadius = new Animated.Value(0);
    var _coreOpacity = new Animated.Value(1);

    var _particlePositions = PARTICLE_COUNT.map(() => new Animated.ValueXY({x: 0, y: 0}));

    this.state.fireworks.push({
      shootingPosition: _shootingPosition,
      shootingColor: _shootingColor,
      particleColor: _particleColor,
      particleRadius: _particleRadius,
      coreOpacity: _coreOpacity,
      particlePositions: _particlePositions
    });

    var _animatedParticles = [
        Animated.timing(_particleRadius, {
          duration: 700,
          toValue: 1
        }),
        Animated.timing(_coreOpacity, {
          duration: 200,
          toValue: 0
        })
    ]

    _movingParticles = _particlePositions.map((particle, i) => {
      var _xy = getXYParticle(PARTICLE_COUNT.length, i, PARTICLE_RADIUS);
      return Animated.timing(particle, {
        duration: 250,
        toValue: _xy
      })
    })

    _animatedParticles = _animatedParticles.concat(_movingParticles);

    Animated.sequence([
      Animated.timing(_shootingPosition, {
        duration: 300,
        toValue: {
          y: e.nativeEvent.locationY,
          x: e.nativeEvent.locationX
        }
      }),
      Animated.parallel(_animatedParticles)
    ]).start(this.removeSelf.bind(this, _shootingPosition));

    _shootingPosition.addListener(this.adjustShootingFill.bind(null, _shootingColor));
    _particleRadius.addListener(this.adjustParticleFill.bind(null, _particleColor));

    this.setState(this.state);
  },

  getFireworks: function() {
    return this.state.fireworks.map((firework, i) => {

      var _shootingFill = firework.shootingColor.interpolate({
        inputRange: [0,1],
        outputRange: SHOOTING_COLORS
      });

      var _particleFill = firework.particleColor.interpolate({
        inputRange: [0,1,2,3,4],
        outputRange: PARTICLE_COLORS
      });

      return (
          <AnimatedGroup 
            x={firework.shootingPosition.x}
            y={firework.shootingPosition.y}
          >
                <AnimatedCircle
                  key={i}
                  opacity={firework.coreOpacity}
                  radius={MORTAR_RADIUS}
                  fill={_shootingFill}
                />
                <Group>
                {
                  PARTICLE_COUNT.map((v, j) => {
                    return <AnimatedCircle
                      x={firework.particlePositions[j].x}
                      y={firework.particlePositions[j].y}
                      scaleX={firework.particleRadius}
                      scaleY={firework.particleRadius}
                      radius={PARTICLE_RADIUS}
                      fill={_particleFill}
                    />
                  })
                }
                </Group>
          </AnimatedGroup>
      );
    })
  },
  render: function() {
    return (
      <View style={styles.container}>
        <TouchableWithoutFeedback onPress={this._handleAddFirework}>
          <View>
            <Surface width={width} height={height}>
              {this.getFireworks()}
            </Surface>
          </View>
        </TouchableWithoutFeedback>
      </View>
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: 'column'
  }
});

function getXYParticle(total, i, radius) {
  var angle = 360/total*i;

  var x = Math.round((radius*2) * Math.cos(angle - (Math.PI/2)));
  var y = Math.round((radius*2) * Math.sin(angle - (Math.PI/2)));

  return {
    x: x,
    y: y
  }
}


//Modified this to use the AnimatedShape we create up above. Thanks Facebook :)

/**
 * Copyright 2013-2014 Facebook, Inc.
 * All rights reserved.
 *
 * This source code is licensed under the BSD-style license found in the
 * LICENSE file in the root directory of this source tree. An additional grant
 * of patent rights can be found in the PATENTS file in the same directory.
 *
 * @providesModule Circle.art
 * @typechecks
 *
 * Example usage:
 * <Circle
 *   radius={10}
 *   stroke="green"
 *   strokeWidth={3}
 *   fill="blue"
 * />
 *
 */
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

AppRegistry.registerComponent('FireworkShooter', () => FireworkShooter);
```