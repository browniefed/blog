---
layout: post
title: "The Shapes of React Native"
date: 2015-09-10 08:21
comments: true
categories: react react-native css shapes
---

# Introduction 

After drawing a bit of inspiration from [The Shapes of CSS](https://css-tricks.com/examples/ShapesOfCSS/) I decided to see if I could remake some of these shapes with a subset of css. If you haven't been on css-tricks check them out, [Chris Coyier](http://chriscoyier.net/) is fantastic!

Of course we have access to `react-art` here so drawing shapes is pretty simple but my goal is to see if I can just use normal `Views` and all of the styles I have at my exposure to make as many shapes as I can off of the Shapes of CSS list.

Some of these are obvious and some of them get a little crazy but most of them are all hacks in the first place!

I'm going on vacation for a month. So this shall be dubbed "One of the more pointless blog posts on my blog written out of sheer tiredness".


{% img http://i.imgur.com/cWR7FKh.gif What am doing with my life %}


## Live Code [https://rnplay.org/apps/58FEmw](https://rnplay.org/apps/58FEmw)

# Key Takeaways

* I wish border-radius worked a little more like the web
* Box Shadow would be nice to have as well.
* Skew transform would be a nice to have. 
* Just use SVGs...

# Shapes

### Square

Pretty simple...

{% img http://i.imgur.com/yNqQt2q.png Yeah what were you expecting %}

```
var Square = React.createClass({
    render: function() {
        return (
            <View style={styles.square} />
        )
    }
});

square: {
    width: 100,
    height: 100,
    backgroundColor: 'red'
}

```

### Rectangle

Nothing too crazy here either

{% img http://i.imgur.com/Eiw8qTZ.png It is a longer square %}

```
var Rectangle = React.createClass({
    render: function() {
        return (
            <View style={styles.rectangle} />
        )
    }
});

rectangle: {
    width: 100 * 2,
    height: 100,
    backgroundColor: 'red'
}

```

### Circle

One note to mention about border radius is that it doesn't work like the web. So if you go more than 50% you'll start forming a weird diamondy shape.

{% img http://i.imgur.com/Monc4Mx.png The circle was invented in 1925 %}

```
var Circle = React.createClass({
    render: function() {
        return (
            <View style={styles.circle} />
        )
    }
})

circle: {
    width: 100,
    height: 100,
    borderRadius: 100/2,
    backgroundColor: 'red'
}

```

### Oval

Border radius wasn't working, lets just do a circle and scale it...

{% img http://i.imgur.com/pHxlyNn.png Not a circle %}

```

var Oval = React.createClass({
    render: function() {
        return (
            <View style={styles.oval} />
        )
    }
});

  oval: {
    width: 100,
    height: 100,
    borderRadius: 50,
    backgroundColor: 'red',
    transform: [
      {scaleX: 2}
    ]
  },
```

###Triangle Up

CSS border triangles still work in React Native.

{% img http://i.imgur.com/cejjWpe.png Pyramid in 2d %}

```
var Triangle = React.createClass({
  render: function() {
    return (
      <View style={[styles.triangle, this.props.style]} />
    )
  }
})

  triangle: {
    width: 0,
    height: 0,
    backgroundColor: 'transparent',
    borderStyle: 'solid',
    borderLeftWidth: 50,
    borderRightWidth: 50,
    borderBottomWidth: 100,
    borderLeftColor: 'transparent',
    borderRightColor: 'transparent',
    borderBottomColor: 'red'
  }

```

Here we get to cheat a bit. You could do this on the web too, but rather than adjust the borders we'll just rotate it.

###Triangle Down

{% img http://i.imgur.com/gwJ9EdU.png Rotate %}
```
var TriangleDown = React.createClass({
  render: function() {
    return (
      <Triangle style={styles.triangleDown}/>
    )
  }

  triangleDown: {
    transform: [
      {rotate: '180deg'}
    ]
  }
```

###Triangle Left

{% img http://i.imgur.com/SlY2Bvf.png Rotate %}

```
var TriangleLeft = React.createClass({
  render: function() {
    return (
      <Triangle style={styles.triangleLeft}/>
    )
  }
})

  triangleLeft: {
    transform: [
      {rotate: '-90deg'}
    ]
  }
```

###Triangle Right

{% img http://i.imgur.com/UTFvVL6.png Konami Code %}

```

var TriangleRight = React.createClass({
  render: function() {
    return (
      <Triangle style={styles.triangleRight}/>
    )
  }
})

  triangleRight: {
    transform: [
      {rotate: '90deg'}
    ]
  },

```

Again we'll cheat here and go for the rotation!

###Triangle Top Left

{% img http://i.imgur.com/aToWUAu.png Pythagorean theory %}
```
var TriangleCorner = React.createClass({
  render: function() {
    return (
      <View style={[styles.triangleCorner, this.props.style]} />
    )
  }
});

  triangleCorner: {
    width: 0,
    height: 0,
    backgroundColor: 'transparent',
    borderStyle: 'solid',
    borderRightWidth: 100,
    borderTopWidth: 100,
    borderRightColor: 'transparent',
    borderTopColor: 'red'
  },

```



###Triangle Top Right

{% img http://i.imgur.com/Ei1GaY4.png sohcahtoa %}


```
var TriangleCornerTopRight = React.createClass({
  render: function() {
    return (
      <TriangleCorner style={styles.triangleCornerTopRight}/>
    )
  }
})

triangleCornerTopRight: {
    transform: [
      {rotate: '90deg'}
    ]
}

```
###Triangle Bottom Left

{% img http://i.imgur.com/tDtSN8B.png ninety degree angle %}

```
var TriangleCornerBottomLeft = React.createClass({
  render: function() {
    return (
      <TriangleCorner style={styles.triangleCornerBottomLeft}/>
    )
  }
})

  triangleCornerBottomLeft: {
    transform: [
      {rotate: '270deg'}
    ]
  },
```

###Triangle Bottom Right

{% img http://i.imgur.com/JbnkwkK.png sick ramp bro %}

```
var TriangleCornerBottomRight = React.createClass({
  render: function() {
    return (
      <TriangleCorner style={styles.triangleCornerBottomRight}/>
    )
  }
})

  triangleCornerBottomRight: {
    transform: [
      {rotate: '180deg'}
    ]
  }

```
###Curved Tail Arrow 

Well we don't have the ability to do pseudo elements but they were just hacks anyway so we'll just create a wrapping `View` with 2 elements and style them.
Now this is not exactly the same, and it's partially due to the way `border-radius` are managed in react-native vs the web but it's closeish.

{% img http://i.imgur.com/Y2IEMxh.png Just use an image %}

```

var CurvedTailArrow = React.createClass({
  render: function() {
    return (
      <View style={styles.curvedTailArrow}>
        <View style={styles.curvedTailArrowTail} />
        <View style={styles.curvedTailArrowTriangle} />
      </View>
    )
  }
})

  curvedTailArrow: {
    backgroundColor: 'transparent',
    overflow: 'visible',
    width: 30,
    height: 25
  },
  curvedTailArrowTriangle: {
    backgroundColor: 'transparent',
    width: 0,
    height: 0,
    borderTopWidth: 9,
    borderTopColor: 'transparent',
    borderRightWidth: 9,
    borderRightColor: 'red',
    borderStyle: 'solid',
    transform: [
      {rotate: '10deg'}
    ],
    position: 'absolute',
    bottom: 9,
    right: 3,
    overflow: 'visible'
  },
  curvedTailArrowTail: {
    backgroundColor: 'transparent',
    position: 'absolute',
    borderBottomColor: 'transparent',
    borderLeftColor: 'transparent',
    borderRightColor: 'transparent',
    borderBottomWidth: 0,
    borderLeftWidth: 0,
    borderRightWidth: 0,
    borderTopWidth: 3,
    borderTopColor: 'red',
    borderStyle: 'solid',
    borderTopLeftRadius: 12,
    top: 1,
    left: 0,
    width: 20,
    height: 20,
    transform: [
      {rotate: '45deg'}
    ]
  }
```

###Trapezoid

The difference with this one is we had to double our width. Why? I don't know.

{% img http://i.imgur.com/Mu3NLyN.png Trapezoid %}

```
var Trapezoid = React.createClass({
  render: function() {
    return (
      <View style={styles.trapezoid} />
    )
  }
})

  trapezoid: {
    width: 200,
    height: 0,
    borderBottomWidth: 100,
    borderBottomColor: 'red',
    borderLeftWidth: 50,
    borderLeftColor: 'transparent',
    borderRightWidth: 50,
    borderRightColor: 'transparent',
    borderStyle: 'solid'
  } 
```

###Parallelogram

If only we had skew. :(
Luckily we have the triangles we created earlier.

{% img http://i.imgur.com/AtXb6rq.png Dont try this at home %}

```

var Parallelogram = React.createClass({
  render: function() {
    return (
      <View style={styles.parallelogram}>
        <TriangleUp style={styles.parallelogramRight} />
        <View style={styles.parallelogramInner} />
        <TriangleDown style={styles.parallelogramLeft} />
      </View>
    )
  }
})


  parallelogram: {
    width: 150,
    height: 100
  },
  parallelogramInner: {
    position: 'absolute',
    left: 0,
    top: 0,
    backgroundColor: 'red',
    width: 150,
    height: 100,
  },
  parallelogramRight: {
    top: 0,
    right: -50,
    position: 'absolute'
  },
  parallelogramLeft: {
    top: 0,
    left: -50,
    position: 'absolute'
  }

```

###Star (6-points)

These Triangles sure are coming in handy.

{% img http://i.imgur.com/XEPeWjV.png This is really ugly, someone should make it look better %}

```
var StarSix = React.createClass({
  render: function() {
    return (
      <View style={styles.starsix}>
        <TriangleUp style={styles.starSixUp} />
        <TriangleDown style={styles.starSixDown} />
      </View>
    )
  }
})

  starsix: {
    width: 100,
    height: 100
  },
  starSixUp: {
    position: 'absolute',
    top: 0,
    left: 0
  },
  starSixDown: {
    position: 'absolute',
    top: 25,
    left: 0
  }

```
###Star (5-points)

Yaye `TriangleUp` is killing it. This one is REALLY hacky with the placement, could use some fine tuning.

{% img http://i.imgur.com/hUvOTUx.png Basically a starfish %}

```

var StarFive = React.createClass({
  render: function() {
    return (
      <View style={styles.starfive}>
        <TriangleUp style={styles.starfiveTop} />
        <View style={styles.starfiveBefore} />
        <View style={styles.starfiveAfter} />
      </View>
    )
  }
})


  starfive: {
    width: 150,
    height: 150,
  },
  starfiveTop: {
    position: 'absolute',
    top: -45,
    left: 37
  },
  starfiveBefore: {
    backgroundColor: 'transparent',
    position: 'absolute',
    left: 0,
    top: 0,
    borderStyle: 'solid',
    borderRightWidth: 100,
    borderRightColor: 'transparent',
    borderBottomWidth: 70,
    borderBottomColor: 'red',
    borderLeftWidth: 100,
    borderLeftColor: 'transparent',
    transform: [
      { rotate: '35deg'}
    ]
  },
  starfiveAfter: {
    backgroundColor: 'transparent',
    position: 'absolute',
    top: 0,
    left: -25,
    width: 0,
    height: 0,
    borderStyle: 'solid',
    borderRightWidth: 100,
    borderRightColor: 'transparent',
    borderBottomWidth: 70,
    borderBottomColor: 'red',
    borderLeftWidth: 100,
    borderLeftColor: 'transparent',
    transform: [
      { rotate: '-35deg'}
    ]
  }
```

###Pentagon

No `TriangleUp` here but we could have used a Corner Triangle with rotate. 

{% img http://i.imgur.com/LEDmb24.png I hate geometry %}

```
var Pentagon = React.createClass({
  render: function() {
    return (
      <View style={styles.pentagon}>
        <View style={styles.pentagonInner} />
        <View style={styles.pentagonBefore} />
      </View>
    )
  }
})

  pentagon: {
    backgroundColor: 'transparent'
  },
  pentagonInner: {
    width: 90,
    borderBottomColor: 'red',
    borderBottomWidth: 0,
    borderLeftColor: 'transparent',
    borderLeftWidth: 18,
    borderRightColor: 'transparent',
    borderRightWidth: 18,
    borderTopColor: 'red',
    borderTopWidth: 50
  },
  pentagonBefore: {
    position: 'absolute',
    height: 0,
    width: 0,
    top: -35,
    left: 0,
    borderStyle: 'solid',
    borderBottomColor: 'red',
    borderBottomWidth: 35,
    borderLeftColor: 'transparent',
    borderLeftWidth: 45,
    borderRightColor: 'transparent',
    borderRightWidth: 45,
    borderTopWidth: 0,
    borderTopColor: 'transparent',
  }
```
###Hexagon

2 Triangles and a square. Everything is just shapes.

{% img http://i.imgur.com/djNMGNg.png Honeycomb %}

```

var Hexagon = React.createClass({
  render: function() {
    return (
      <View style={styles.hexagon}>
        <View style={styles.hexagonInner} />
        <View style={styles.hexagonBefore} />
        <View style={styles.hexagonAfter} />
      </View>
    )
  }
})

  hexagon: {
    width: 100,
    height: 55
  },
  hexagonInner: {
    width: 100,
    height: 55,
    backgroundColor: 'red'
  },
  hexagonAfter: {
    position: 'absolute',
    bottom: -25,
    left: 0,
    width: 0,
    height: 0,
    borderStyle: 'solid',
    borderLeftWidth: 50,
    borderLeftColor: 'transparent',
    borderRightWidth: 50,
    borderRightColor: 'transparent',
    borderTopWidth: 25,
    borderTopColor: 'red'
  },
  hexagonBefore: {
    position: 'absolute',
    top: -25,
    left: 0,
    width: 0,
    height: 0,
    borderStyle: 'solid',
    borderLeftWidth: 50,
    borderLeftColor: 'transparent',
    borderRightWidth: 50,
    borderRightColor: 'transparent',
    borderBottomWidth: 25,
    borderBottomColor: 'red'

  }

```

###Octagon

I attempted copied the css on this one but it required setting a background color, so I did 4 bars and just rotated them. Slightly more markup but this is just for fun.

{% img http://i.imgur.com/i5drMtB.png Stop! %}

```
var Octagon = React.createClass({
  render: function() {
    return (
      <View style={styles.octagon}>
        <View style={[styles.octagonUp, styles.octagonBar]} />
        <View style={[styles.octagonFlat, styles.octagonBar]} />
        <View style={[styles.octagonLeft, styles.octagonBar]} />
        <View style={[styles.octagonRight, styles.octagonBar]} />
      </View>
    )
  }
})



  octagon: {},
  octagonBar: {
    width: 42,  
    height: 100,
    backgroundColor: 'red'
  },
  octagonUp: {},
  octagonFlat: {
    position: 'absolute',
    top: 0,
    left: 0,
    transform: [
      {rotate: '90deg'}
    ]
  },
  octagonLeft: {
    position: 'absolute',
    top: 0,
    left: 0,
    transform: [
      {rotate: '-45deg'}
    ]
  },
  octagonRight: {
    position: 'absolute',
    top: 0,
    left: 0,
    transform: [
      {rotate: '45deg'}
    ]
  }
```

###Heart 

This one is easy since well I already had it done for my previous tutorial.

{% img http://i.imgur.com/uBAv2eJ.png in the name of love %}

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
###Infinity 

Width and border radius all work oddly together. So baby infinity? Scale it up if you want it bigger.

{% img http://i.imgur.com/7Ykpa06.png This could be animated and you would have no idea %}

```
var Infinity = React.createClass({
  render: function() {
    return (
      <View style={styles.infinity}>
        <View style={styles.infinityBefore} />
        <View style={styles.infinityAfter} />
      </View>   
    )
  }
})

  infinity: {
    width: 80,
    height: 100,
  },
  infinityBefore: {
    position: 'absolute',
    top: 0,
    left: 0,
    width: 0,
    height: 0,
    borderWidth: 20,
    borderColor: 'red',
    borderStyle: 'solid',
    borderTopLeftRadius: 50,
    borderTopRightRadius: 50,
    borderBottomRightRadius: 50,
    borderBottomLeftRadius: 0,
    transform: [
      {rotate: '-135deg'}
    ]
  },
  infinityAfter: {
    position: 'absolute',
    top: 0,
    right: 0,
    width: 0,
    height: 0,
    borderWidth: 20,
    borderColor: 'red',
    borderStyle: 'solid',
    borderTopLeftRadius: 50,
    borderTopRightRadius: 0,
    borderBottomRightRadius: 50,
    borderBottomLeftRadius: 50,
    transform: [
      {rotate: '-135deg'}
    ]
  }
```
###Diamond Square 

This was more than just a rotated square. Am I missing something?

{% img http://i.imgur.com/gAL9dfq.png Not a blood diamond %}

```

var Diamond = React.createClass({
  render: function() {
    return (
      <View style={styles.diamond} />
    )
  }
})

  diamond:{
    width: 50,
    height: 50,
    backgroundColor: 'red',
    transform: [
      {rotate: '45deg'}
    ]    
  }

```


###Diamond Shield 

Just 2 triangles, thought this one was going to be harder.

{% img http://i.imgur.com/4S6FDMo.png also just a kite %}

```

var DiamondShield = React.createClass({
  render: function() {
    return (
      <View style={styles.diamondShield}>
        <View style={styles.diamondShieldTop} />
        <View style={styles.diamondShieldBottom} />
      </View>
    )
  }
})

  diamondShield: {
    width: 100,
    height: 100
  },
  diamondShieldTop: {
    width: 0,
    height: 0,
    borderTopWidth: 50,
    borderTopColor: 'transparent',
    borderLeftColor: 'transparent',
    borderLeftWidth: 50,
    borderRightColor: 'transparent',
    borderRightWidth: 50,
    borderBottomColor: 'red',
    borderBottomWidth: 20,
  },
  diamondShieldBottom: {
    width: 0,
    height: 0,
    borderTopWidth: 70,
    borderTopColor: 'red',
    borderLeftColor: 'transparent',
    borderLeftWidth: 50,
    borderRightColor: 'transparent',
    borderRightWidth: 50,
    borderBottomColor: 'transparent',
    borderBottomWidth: 50,
  }
```

###Diamond Narrow

Another 2 triangles that could have been the same and rotated. This way works too.

{% img http://i.imgur.com/cRwI61S.png Diamond on a diet %}

```
var DiamondNarrow = React.createClass({
  render: function() {
    return (
      <View style={styles.diamondNarrow}>
        <View style={styles.diamondNarrowTop} />
        <View style={styles.diamondNarrowBottom} />
      </View>
    )
  }
})

  diamondNarrow: {
    width: 100,
    height: 100
  },
  diamondNarrowTop: {
    width: 0,
    height: 0,
    borderTopWidth: 50,
    borderTopColor: 'transparent',
    borderLeftColor: 'transparent',
    borderLeftWidth: 50,
    borderRightColor: 'transparent',
    borderRightWidth: 50,
    borderBottomColor: 'red',
    borderBottomWidth: 70,  
  },
  diamondNarrowBottom: {
    width: 0,
    height: 0,
    borderTopWidth: 70,
    borderTopColor: 'red',
    borderLeftColor: 'transparent',
    borderLeftWidth: 50,
    borderRightColor: 'transparent',
    borderRightWidth: 50,
    borderBottomColor: 'transparent',
    borderBottomWidth: 50, 
  }

```
###Cut Diamond

The top could have been used for the octagon, I chose a different way though.

{% img http://i.imgur.com/yZNHZP0.png Now that is a diamond %}

```


var CutDiamond = React.createClass({
  render: function() {
    return (
      <View style={styles.cutDiamond}>
        <View style={styles.cutDiamondTop} />
        <View style={styles.cutDiamondBottom} />
      </View>
    )
  }
})

  cutDiamond: {
    width: 100,
    height: 100,
  },
  cutDiamondTop: {
    width: 100,
    height: 0,
    borderTopWidth: 0,
    borderTopColor: 'transparent',
    borderLeftColor: 'transparent',
    borderLeftWidth: 25,
    borderRightColor: 'transparent',
    borderRightWidth: 25,
    borderBottomColor: 'red',
    borderBottomWidth: 25, 
  },
  cutDiamondBottom: {
    width: 0,
    height: 0,
    borderTopWidth: 70,
    borderTopColor: 'red',
    borderLeftColor: 'transparent',
    borderLeftWidth: 50,
    borderRightColor: 'transparent',
    borderRightWidth: 50,
    borderBottomColor: 'transparent',
    borderBottomWidth: 0, 
  }
```

###Egg

Circular things are hard to do in RN. This is eggish.

{% img http://i.imgur.com/0v5tH4x.png Her? %}

{% img http://i.imgur.com/27Rbo3x.gif Egg? %}

```
var Egg = React.createClass({
  render: function() {
    return (
      <View style={styles.egg} />
    )
  }
})

  egg: {
    width: 126,
    height: 180,
    backgroundColor: 'red',
    borderTopLeftRadius: 108,
    borderTopRightRadius: 108,
    borderBottomLeftRadius: 95,
    borderBottomRightRadius: 95
  }
```

###Pac-Man

This one is so simple but always so fun.

{% img http://i.imgur.com/TZHjuxw.png Pixels was a teribel movie %}

```
var PacMan = React.createClass({
  render: function() {
    return (
      <View style={styles.pacman}/>
    )
  }
})

  pacman: {
    width: 0,
    height: 0,
    borderTopWidth: 60,
    borderTopColor: 'red',
    borderLeftColor: 'red',
    borderLeftWidth: 60,
    borderRightColor: 'transparent',
    borderRightWidth: 60,
    borderBottomColor: 'red',
    borderBottomWidth: 60, 
    borderTopLeftRadius: 60,
    borderTopRightRadius: 60,
    borderBottomRightRadius: 60,
    borderBottomLeftRadius: 60
  }
```

###Talk Bubble

This one is also simple, triangle and a rounded square.

{% img http://i.imgur.com/1LIwGEQ.png Perfect for your billion dollar slack clone %}

```
var TalkBubble = React.createClass({
  render: function() {
    return (
      <View style={styles.talkBubble}>
        <View style={styles.talkBubbleSquare} />
        <View style={styles.talkBubbleTriangle} />
      </View>   
    )
  }
})

  talkBubble: {
    backgroundColor: 'transparent'
  },
  talkBubbleSquare: {
    width: 120,
    height: 80,
    backgroundColor: 'red',
    borderRadius: 10
  },
  talkBubbleTriangle: {
    position: 'absolute',
    left: -26,
    top: 26,
    width: 0,
    height: 0,
    borderTopColor: 'transparent',
    borderTopWidth: 13,
    borderRightWidth: 26,
    borderRightColor: 'red',
    borderBottomWidth: 13,
    borderBottomColor: 'transparent'
  }
```

###12 Point Burst

I will admit this one confused be a little bit, then I realized it's just a couple of rotated squares.

{% img http://i.imgur.com/FHx0WVH.png NOW 90% OFF!! %}

```
var TwelvePointBurst = React.createClass({
  render: function() {
    return (
      <View style={styles.twelvePointBurst}>
        <View style={styles.twelvePointBurstMain} />
        <View style={styles.twelvePointBurst30} />
        <View style={styles.twelvePointBurst60} />
      </View>   
    )
  }
})


 twelvePointBurst: {},
  twelvePointBurstMain: {
    width: 80,
    height: 80,
    backgroundColor: 'red'
  },
  twelvePointBurst30: {
    width: 80, 
    height: 80,
    position: 'absolute',
    backgroundColor: 'red',
    top: 0,
    right: 0,
    transform: [
      {rotate: '30deg'}
    ]
  },
  twelvePointBurst60: {
    width: 80, 
    height: 80,
    position: 'absolute',
    backgroundColor: 'red',
    top: 0,
    right: 0,
    transform: [
      {rotate: '60deg'}
    ]
  },

```

###8 Point Burst 
Just like the 12, but one less square and different rotations. Only thing here is because the pseudo element was positionined relative to the first 20 degree rotation and ours isn't we'll just bump it up to 155.

{% img http://i.imgur.com/IITGOMB.png Sun %}

```
var EightPointBurst = React.createClass({
  render: function() {
    return (
      <View style={styles.eightPointBurst}>
        <View style={styles.eightPointBurst20} />
        <View style={styles.eightPointBurst155} />
      </View>   
    )
  }
})

  eightPointBurst: {},
  eightPointBurst20: {
    width: 80, 
    height: 80,
    backgroundColor: 'red',
    transform: [
      {rotate: '20deg'}
    ]
  },
  eightPointBurst155: {
    width: 80, 
    height: 80,
    position: 'absolute',
    backgroundColor: 'red',
    top: 0,
    left: 0,
    transform: [
      {rotate: '155deg'}
    ]
  },

```


###Yin Yang

This one I don't like because you can't accomplish it without setting a background. Ohwell.
Also weird border issue causing outlines.

{% img http://i.imgur.com/z9cUqaz.png Yin and Yang and see through background borders %}

```

var YinYang = React.createClass({
  render: function() {
    return (
      <View style={styles.yinyang}>
        <View style={styles.yinyangMain} />
        <View style={styles.yinyangBefore} />
        <View style={styles.yinyangAfter} />
      </View>   
    )
  }
})

  yinyang: {

  },
  yinyangMain: {
    width: 100,
    height: 100,
    borderColor: 'red',
    borderTopWidth: 2,
    borderLeftWidth: 2,
    borderBottomWidth: 50,
    borderRightWidth: 2,
    borderRadius: 50
  },
  yinyangBefore: {
    position: 'absolute',
    top: 24,
    left: 0,
    borderColor: 'red',
    borderWidth: 24,
    borderRadius: 30,
  },
  yinyangAfter: {
    position: 'absolute',
    top: 24,
    right: 2,
    backgroundColor: 'red',
    borderColor: 'white',
    borderWidth: 25,
    borderRadius: 30,
  }
```

###Badge Ribbon

Remember, always add `backgroundColor: 'transparent'` when you are overlapping things.

{% img http://i.imgur.com/3V4K2B3.png Well I did get first place %}

```
var BadgeRibbon = React.createClass({
  render: function() {
    return (
      <View style={styles.badgeRibbon}>
        <View style={styles.badgeRibbonCircle} />
        <View style={styles.badgeRibbonNeg140} />
        <View style={styles.badgeRibbon140} />
      </View>   
    )
  }
})
  badgeRibbonCircle: {
    width: 100,
    height: 100,
    backgroundColor: 'red',
    borderRadius: 50
  },
  badgeRibbon140: {
    backgroundColor:'transparent',
    borderBottomWidth: 70,
    borderBottomColor: 'red',
    borderLeftWidth: 40,
    borderLeftColor: 'transparent',
    borderRightWidth: 40,
    borderRightColor: 'transparent',
    position: 'absolute',
    top: 70,
    right: -10,
    transform: [
      {rotate: '140deg'}
    ]
  },
  badgeRibbonNeg140: {
    backgroundColor:'transparent',
    borderBottomWidth: 70,
    borderBottomColor: 'red',
    borderLeftWidth: 40,
    borderLeftColor: 'transparent',
    borderRightWidth: 40,
    borderRightColor: 'transparent',
    position: 'absolute',
    top: 70,
    left: -10,
    transform: [
      {rotate: '-140deg'}
    ]
  }
```

###Space Invader

 `WUTTTTTTTTTTT`

###TV Screen

Stupid border radius making this one hard. We'll just use a bunch of ovals. 

{% img http://i.imgur.com/ffJdfqM.png CRT %}

```
var TvScreen = React.createClass({
  render: function() {
    return (
      <View style={styles.tvscreen}>
        <View style={styles.tvscreenMain} />
        <View style={styles.tvscreenTop} />
        <View style={styles.tvscreenBottom} />
        <View style={styles.tvscreenLeft} />
        <View style={styles.tvscreenRight} />

      </View>   
    )
  }
})

  tvscreen: {},
  tvscreenMain: {
    width: 150,
    height: 75,
    backgroundColor: 'red',
    borderTopLeftRadius: 15,
    borderTopRightRadius: 15,
    borderBottomRightRadius: 15,
    borderBottomLeftRadius: 15,
  },
  tvscreenTop: {
    width: 73,
    height: 70,
    backgroundColor: 'red',
    position: 'absolute',
    top: -26,
    left: 39,
    borderRadius: 35,
    transform: [
      {scaleX: 2},
      {scaleY: .5}
    ]
  },
  tvscreenBottom: {
    width: 73,
    height: 70,
    backgroundColor: 'red',
    position: 'absolute',
    bottom: -26,
    left: 39,
    borderRadius: 35,
    transform: [
      {scaleX: 2},
      {scaleY: .5}
    ]
  },
  tvscreenLeft: {
    width: 20,
    height: 38,
    backgroundColor: 'red',
    position: 'absolute',
    left: -7,
    top: 18,
    borderRadius: 35,
    transform: [
      {scaleX: .5},
      {scaleY: 2},
    ]
  },
  tvscreenRight: {
    width: 20,
    height: 38,
    backgroundColor: 'red',
    position: 'absolute',
    right: -7,
    top: 18,
    borderRadius: 35,
    transform: [
      {scaleX: .5},
      {scaleY: 2},
    ]
  },
```

###Chevron

Once again we don't have skew, but we'll use triangles. Also magical negative scale to flip stuff around!

{% img http://i.imgur.com/HEfbLbS.png get techron with chevron %}

```

var Chevron = React.createClass({
  render: function() {
    return (
      <View style={styles.chevron}>
        <View style={styles.chevronMain} />
        <View style={[styles.chevronTriangle, styles.chevronTopLeft]} />
        <View style={[styles.chevronTriangle, styles.chevronTopRight]} />
        <View style={[styles.chevronTriangle, styles.chevronBottomLeft]} />
        <View style={[styles.chevronTriangle, styles.chevronBottomRight]} />
      </View>   
    )
  }
})


  chevron: {
    width: 150,
    height: 50
  },
  chevronMain: {
    width: 150,
    height: 50,
    backgroundColor: 'red'
  },
  chevronTriangle: {
    backgroundColor: 'transparent',
    borderTopWidth: 20,
    borderRightWidth: 0,
    borderBottomWidth: 0,
    borderLeftWidth: 75,
    borderTopColor: 'transparent',
    borderBottomColor: 'transparent',
    borderRightColor: 'transparent',
    borderLeftColor: 'red',
  },
  chevronTopLeft: {
    position: 'absolute',
    top: -20,
    left: 0
  },
  chevronTopRight: {
    position: 'absolute',
    top: -20,
    right: 0,
    transform: [
      {scaleX: -1}
    ]
  },
  chevronBottomLeft: {
    position: 'absolute',
    bottom: -20,
    left: 0,
    transform: [
      {scale: -1 }
    ]
  },     
  chevronBottomRight: {
    position: 'absolute',
    bottom: -20,
    right: 0,
    transform: [
      {scaleY: -1}
    ]
  }
```

###Magnifying Glass

Border around a circle with a stick. Nothing to it.

{% img http://i.imgur.com/1aCNZLk.png Blow bubbles %}

```
var MagnifyingGlass = React.createClass({
  render: function() {
    return (
      <View style={styles.magnifyingGlass}>
        <View style={styles.magnifyingGlassCircle} />
        <View style={styles.magnifyingGlassStick} />
      </View>   
    )
  }
})

  magnifyingGlass: {

  },
  magnifyingGlassCircle: {
    width: 100,
    height: 100,
    borderRadius: 50,
    borderWidth: 15,
    borderColor: 'red'
  },
  magnifyingGlassStick: {
    position: 'absolute',
    right: -20,
    bottom: -10,
    backgroundColor: 'red',
    width: 50,
    height: 10,
    transform: [
      {rotate: '45deg'}
    ]
```
###Facebook Icon 

This one seems appropriate but couldn't get it to work well. I attempted it and failed.

{% img http://i.imgur.com/Y9lyxN7.png React Native brought to you by %}

```
var Facebook = React.createClass({
  render: function() {
    return (
      <View style={styles.facebook}>
        <View style={styles.facebookMain}>          
          <View style={styles.facebookCurve} />
          <View style={styles.facebookBefore} />
          <View style={styles.facebookAfter} />
          <View style={styles.facebookRedCover} />
        </View>
      </View>   
    )
  }
})

  facebook: {
    width: 100,
    height: 110,
  },
  facebookMain: {
    backgroundColor: 'red',
    width: 100,
    height: 110,
    borderRadius: 5,
    borderColor: 'red',
    borderTopWidth: 15,
    borderLeftWidth: 15,
    borderRightWidth: 15,
    borderBottomWidth: 0,
    overflow: 'hidden'

  },
  facebookRedCover: {
    width: 10,
    height: 20,
    backgroundColor: 'red',
    position: 'absolute',
    right: 0,
    top: 5
  },
  facebookCurve: {
    width: 50,
    borderWidth: 20,
    borderTopWidth: 20,
    borderTopColor: 'white',
    borderBottomColor: 'transparent',
    borderLeftColor: 'white',
    borderRightColor: 'transparent',
    borderRadius: 20,
    position: 'absolute',
    right: -8,
    top: 5
  },
  facebookBefore: {
    position: 'absolute',
    backgroundColor: 'white',
    width: 20,
    height: 70,
    bottom: 0,
    right: 22,
  },
  facebookAfter: {
    position: 'absolute',
    width: 55,
    top: 50,
    height: 20,
    backgroundColor: 'white',
    right: 5
  }
``` 
###Moon 

Box shadow...

###Flag 

The one on css-tricks inferred a background, we'll just flip it around and say the center is transparent and the outer triangles are red.

{% img http://i.imgur.com/7AMJ3sj.png Have they ever seen a flag? %}

```

var Flag = React.createClass({
  render: function() {
    return (
      <View style={styles.flag}>
        <View style={styles.flagTop} />
        <View style={styles.flagBottom} />
      </View>   
    )
  }
})

  flag: {},
  flagTop: {
    width: 110,
    height: 56,
    backgroundColor: 'red',
  },
  flagBottom: {
    position: 'absolute',
    left: 0,
    bottom: 0,
    width: 0,
    height: 0,
    borderBottomWidth: 13,
    borderBottomColor: 'transparent',
    borderLeftWidth: 55,
    borderLeftColor: 'red',
    borderRightWidth: 55,
    borderRightColor: 'red'
  }
```

###Cone 

Had to modify the css on this one a bit to get the same look, 70 => 55.

{% img http://i.imgur.com/04f26Kl.png needs more icecream %}

```
var Cone = React.createClass({
  render: function() {
    return (
      <View style={styles.cone} />
    )
  }
})

  cone: {
    width: 0,
    height: 0,
    borderLeftWidth: 55,
    borderLeftColor: 'transparent',
    borderRightWidth: 55,
    borderRightColor: 'transparent',
    borderTopWidth: 100,
    borderTopColor: 'red',
    borderRadius: 55
  }
```

###Cross

More of a plus then a cross.

{% img http://i.imgur.com/0IerLuP.png across from where? %}

{% img http://i.imgur.com/zfWtpTN.gif heh %}

```
var Cross = React.createClass({
  render: function() {
    return (
      <View style={styles.cross}>
        <View style={styles.crossUp} />
        <View style={styles.crossFlat} />
      </View>   
    )
  }
})

  cross: {

  },
  crossUp: {
    backgroundColor: 'red',
    height: 100,
    width: 20
  },
  crossFlat: {
    backgroundColor: 'red',
    height: 20,
    width: 100,
    position: 'absolute',
    left: -40,
    top: 40
  }
```

###Base 

Base... Home .. Home Base, whichever all the same.

{% img http://i.imgur.com/LGQEIvS.png that's all folks %}

```
var Base = React.createClass({
  render: function() {
    return (
      <View style={styles.base}>
        <View style={styles.baseTop} />
        <View style={styles.baseBottom} />
      </View>   
    )
  }
})
  base: {

  },
  baseTop: {
    borderBottomWidth: 35,
    borderBottomColor: 'red',
    borderLeftWidth: 50,
    borderLeftColor: 'transparent',
    borderRightWidth: 50,
    borderRightColor: 'transparent',
    height: 0,
    width: 0,
    left: 0,
    top: -35,
    position: 'absolute',
  },
  baseBottom: {
    backgroundColor: 'red',
    height: 55,
    width: 100
  }
```


#Final 

Wow what a fun waste of time. Modeling React Native after the web spec is of course a great idea, I just wish it conformed a little nicer on border radius.

Also I hate geometry now.

## Live Code [https://rnplay.org/apps/58FEmw](https://rnplay.org/apps/58FEmw)

I'm not posting the full code here because it's just too long.


{% img http://i.imgur.com/cWR7FKh.gif Stay in school kids %}
