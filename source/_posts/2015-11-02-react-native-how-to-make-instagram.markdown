---
layout: post
title: "React Native - How to make Instagram"
date: 2015-11-02 04:48
comments: true
categories: react-native react instagram gl gl-react-native
facebook:
    image: http://i.imgur.com/AaPPZsC.png
twitter_card:
    creator: browniefed
    type: summary_large_image
    image: http://i.imgur.com/AaPPZsC.png
---

## Intro
Instagram is a fantastic app and a great concept to model after for learning fragment shaders. We won't get too deep into fragment shaders but I'll take a little bit about what they are and point you to some resources.

We'll take advantage of the awesome [gl-react-native](https://github.com/ProjectSeptemberInc/gl-react-native) component library by [Gaëtan Renaudeau](https://twitter.com/greweb).

There are some fantastic resources on basic concepts of fragment shaders, check them out below. Much of the fragment shader code we'll write is taken from there and or slightly modified! I am not an expert on this stuff, I'm just playing around.

* GL React documentation: [https://projectseptemberinc.gitbooks.io/gl-react/content/](https://projectseptemberinc.gitbooks.io/gl-react/content/)
* Great explanation of fragment shader effects: [https://github.com/yulu/GLtext](https://github.com/yulu/GLtext)
* Some Instagram fragment shader pre-sets: [https://github.com/yulu/Instagram_Filter/tree/master/res/raw](https://github.com/yulu/Instagram_Filter/tree/master/res/raw)
* More Instagram pre-sets however they're in CSS: [http://una.im/CSSgram/](http://una.im/CSSgram/)

## What are we building

{% img http://i.imgur.com/IQDx6Ls.gif More saturation please %}

<!-- more -->

## Concept

We'll write a fragment shader that takes various values and adjust a child image. We'll throw some sliders on there so the user can control it.


## Disclaimer

I have no clue if this is the correct way to do this. Once again, I'm just playing around. The presets that I linked to above may not output the exact filter you are expecting. Yes you may have to adjust the shader code to make it perfectly match whatever Instagram actually does. Don't ask me to do this for you.

Want to save off what you did? Check out [https://github.com/jsierles/react-native-view-snapshot](https://github.com/jsierles/react-native-view-snapshot) or check out [https://github.com/BradLarson/GPUImage](https://github.com/BradLarson/GPUImage) for some powerful image manipulations on iOS!


## Install

You'll need to do `npm install gl-react-native` and also add it to Xcode. `gl-react-native` has instructions on how to do that in the README at [https://github.com/ProjectSeptemberInc/gl-react-native](https://github.com/ProjectSeptemberInc/gl-react-native).

## Setup

```
var React = require('react-native');
var GL = require('gl-react-native');

var {
  AppRegistry,
  StyleSheet,
  Text,
  Image,
  View,
  ScrollView,
  SliderIOS
} = React;
```

Nothing too special here.


## Create a GL.View

```
var Instagram = GL.createComponent(
  ({ children, ...rest }) =>
  <GL.View
    {...rest}
    shader={shaders.instagram}
    uniforms={{  }}>
    <GL.Uniform name="tex">{children}</GL.Uniform>
  </GL.View>
, { displayName: "Instagram" });

```

`GL.createComponent` takes a function that returns and creates everything you need to render. It then provides the props to the function when it wants to render.

`GL.View` is what receives the properties and the shader. The `GL.Uniform` is given a name that will be provided to the shader. The main purpose is to provide a texture to grab pixels from to feed the shader.

If you wanted a blank canvas to render arbitrary shaders then you would not need the `GL.Uniform`!

## Create an empty Shader

```
const shaders = GL.Shaders.create({
  instagram: {
    frag: `
      void main() {

      }
    `
  }
});

```

We call `GL.Shaders.create`. At some point `gl-react-native` may support Vertex shaders instead of just Fragment shaders. So we scope our `instagram` shader with another key `frag` and use ES2015/ES6 template strings so we can quickly edit and manipulate the shader rather than having to deal with quotes.

## Render Empty

```
var rn_instagram = React.createClass({
  getInitialState: function() {
    return {
      width:0,
      height: 0,
    };
  },
  renderWithDimensions: function(layout) {
    var {
      width,
      height
    } = layout.nativeEvent.layout;
    this.setState({
      width,
      height
    })
  },
  getImage: function() {
    return (
      <Instagram 
        width={this.state.width}
        height={this.state.height}
      >
        <Image
          source={{uri: 'http://i.imgur.com/dSIa9jl.jpg'}}
          style={styles.cover}
          resizeMode="cover"
        />
      </Instagram>

    )
  },
  render: function() {
    return (
      <View style={styles.container}>
        <View style={styles.container} onLayout={this.renderWithDimensions}>
          { this.state.width ? this.getImage() : null}
        </View>
      </View>
    )
  }
});
```
One limitation of `gl-react-native` is that width/height are almost always required. I say almost because I don't know for sure, but so far it seems to always be required.

So what that means is we need to create a container with `flex:1` so we can then use the `onLayout` function to get the measured width/height of whatever `flex:1` translates to. Once it is set in our `state` then we can render our `Instagram` component with our `Image`. 

`Image` has a `cover` class and `resizeMode` set to cover.
Our cover class looks like so

```
  cover: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0
  }
```
This will have the image cover and resize to fit the space provided.


## Create the master shader

```
const shaders = GL.Shaders.create({
  instagram: {
    frag: `
      precision highp float;
      varying vec2 uv;
      uniform sampler2D tex;

      uniform float saturation;
      uniform float brightness;
      uniform float contrast;
      uniform float hue;
      uniform float gray;
      uniform float sepia;
      uniform float mixFactor;

      const vec3 W = vec3(0.2125, 0.7154, 0.0721);
      const mat3 rgb2yiq = mat3(0.299, 0.587, 0.114, 0.595716, -0.274453, -0.321263, 0.211456, -0.522591, 0.311135);
      const mat3 yiq2rgb = mat3(1.0, 0.9563, 0.6210, 1.0, -0.2721, -0.6474, 1.0, -1.1070, 1.7046);
      const vec3 SEPIA = vec3(1.2, 1.0, 0.8);

      vec3 BrightnessContrastSaturation(vec3 color, float brt, float con, float sat)
      {
        vec3 black = vec3(0., 0., 0.);
        vec3 middle = vec3(0.5, 0.5, 0.5);
        float luminance = dot(color, W);
        vec3 gray = vec3(luminance, luminance, luminance);
        
        vec3 brtColor = mix(black, color, brt);
        vec3 conColor = mix(middle, brtColor, con);
        vec3 satColor = mix(gray, conColor, sat);
        return satColor;
      }

      vec3 multiplyBlender(vec3 Color, vec3 filter){
        vec3 filter_result;
        float luminance = dot(filter, W);
        
        if(luminance < 0.5)
          filter_result = 2. * filter * Color;
        else
          filter_result = Color;
            
        return filter_result;
      }

      vec3 ovelayBlender(vec3 Color, vec3 filter){
        vec3 filter_result;

        float luminance = dot(filter, W);
        
        if(luminance < 0.5)
          filter_result = 2. * filter * Color;
        else
          filter_result = 1. - (1. - (2. *(filter - 0.5)))*(1. - Color);
          
        return filter_result;
      }

      vec3 applyHue(vec3 Color, float h) {
        vec3 yColor = rgb2yiq * Color;
        float originalHue = atan(yColor.b, yColor.g);
        float finalHue = originalHue + (h);
        float chroma = sqrt(yColor.b*yColor.b+yColor.g*yColor.g);
        vec3 yFinalColor = vec3(yColor.r, chroma * cos(finalHue), chroma * sin(finalHue));
        return vec3(yiq2rgb*yFinalColor);
      }

      vec3 applyGray(vec3 Color, float g) {
        float gray = dot(Color, vec3(0.299, 0.587, 0.114));
        return mix(Color, vec3(gray, gray, gray), g);
      }

      vec3 applySepia(vec3 Color, float s) {
        float gray = dot(Color, vec3(0.299, 0.587, 0.114));
        return mix(Color, vec3(gray) * SEPIA, s);
      }


      void main() {
        vec2 st = uv.st;
        vec3 irgb = texture2D(tex, st).rgb;
        vec3 filter = texture2D(tex, st).rgb;

        vec3 bcs_result = BrightnessContrastSaturation(irgb, brightness, contrast, saturation);
        vec3 hue_result = applyHue(bcs_result, hue);
        vec3 sepia_result = applySepia(hue_result, sepia);
        vec3 gray_result = applyGray(sepia_result, gray);

        vec3 after_filter = mix(gray_result, multiplyBlender(gray_result, filter), mixFactor);
        
        gl_FragColor = vec4(after_filter, 1.);
      }
    `
  }
});

```

I wont' go too indepth here since I don't know a ton about what is happening. But quick explanation.

```
    varying vec2 uv;
    uniform sampler2D tex;

    uniform float saturation;
    uniform float brightness;
    uniform float contrast;
    uniform float hue;
    uniform float gray;
    uniform float sepia;
    uniform float mixFactor;
```
When we declare `uniform` in our shader it means that it is a value that is coming in from the outside. In our case from the JS world to the Obj-C world and into our shader.

We need to define it's type, in our case it's a `float` for most input values.

The `varying vec2 uv` is what I believe is the outside world providing the texture coordinates of what  is currently being processed. That way you can do specific things based on the coordinates you are at... like transforming a pixel color from one thing to another, adding things like vignettes, or whatever else you want to do.

In our `texture` case we receive a `sampler2D`. This has a bunch of data that allows us to extract a `rgb` out of it using our coordinates from above.
This is where we get the `rgb` value to manipulate based upon our shader.

We get that `rgb` value here `texture2D(tex, st).rgb;`. Which is assigned to a `vec3` which just is an arbitrary container of `3` values. In JavaScript just think of it as an array like `[1,2,3]` but can be referenced like an object with `.` notation.


```
void main() {
    vec2 st = uv.st;
    vec3 irgb = texture2D(tex, st).rgb;
    vec3 filter = texture2D(tex, st).rgb;

    vec3 bcs_result = BrightnessContrastSaturation(irgb, brightness, contrast, saturation);
    vec3 hue_result = applyHue(bcs_result, hue);
    vec3 sepia_result = applySepia(hue_result, sepia);
    vec3 gray_result = applyGray(sepia_result, gray);

    vec3 after_filter = mix(gray_result, multiplyBlender(gray_result, filter), mixFactor);

    gl_FragColor = vec4(after_filter, 1.);
}
```

Our main function is called and is where things start. We extract our coordinates from `uv`. Get our `rgb` value at those specific coordiantes.

Then pass it through our functions. Each function takes one or more of our `uniform` values that gets passed in. It then returns a `vec3` which is just an `rgb` color value. That color then gets passed into the next function.

We essentially just keep passing and mutating an `rgb` color value until the end. 

Eventually `gl_FragColor` is where we spit out our final color for that specific coordinate that our shader was called with.


## Add State

```
  getInitialState: function() {
    return {
      width:0,
      height: 0,
      saturation: 1,
      brightness: 1,
      contrast: 1,
      hue: 0,
      sepia: 0,
      gray: 0,
      mixFactor: 0
    };
  },
```

We should fix up our state to have all the values that our shader needs. In our case we just make up a values for each shader and set random defaults.

## Fix the GL.View

```
var Instagram = GL.createComponent(
  ({ brightness, saturation, contrast, hue, gray, sepia, mixFactor, children, ...rest }) =>
  <GL.View
    {...rest}
    shader={shaders.instagram}
    uniforms={{ brightness, saturation, contrast, hue, gray, sepia, mixFactor }}>
    <GL.Uniform name="tex">{children}</GL.Uniform>
  </GL.View>
, { displayName: "Instagram" });
```

Now that You can see we now pass in all the necessary uniforms. If they exist here they must exist in your shader. If they exist in your shader and aren't used you will get an error! Or vice versa. Ultimately if you forget something you'll know because your shader won't compile and you'll get a red error screen like you may be used to.


```
getImage: function() {
    return (
      <Instagram 
        brightness={this.state.brightness}
        saturation={this.state.saturation}
        contrast={this.state.contrast}
        hue={this.state.hue}
        gray={this.state.gray}
        sepia={this.state.sepia}
        mixFactor={this.state.mixFactor}
        width={this.state.width}
        height={this.state.height}
      >
        <Image
          source={{uri: 'http://i.imgur.com/dSIa9jl.jpg'}}
          style={styles.cover}
          resizeMode="cover"
        />
      </Instagram>

    )
  },
```

Pass in our state to the `GL.View` we created. We technically could have just used the `...` spread operator here but I'm being explicity for the sake of this tutorial.

## Add Controls

```
        <ScrollView style={styles.container}>
          <View>
            <Text>Blend Factor: {this.state.mixFactor}</Text>
            <SliderIOS
              value={this.state.mixFactor}
              minimumValue={0}
              maximumValue={2}
              onValueChange={(mixFactor) => this.setState({mixFactor})}
            />
          </View>
          <View>
            <Text>Brightness: {this.state.brightness}</Text>
            <SliderIOS
              value={this.state.brightness}
              minimumValue={0}
              maximumValue={3}
              onValueChange={(brightness) => this.setState({brightness})}
            />
          </View>
        //OTHER CONTROLS
        </ScrollView>
```

Nothing too crazy here either. We just setup our controls. When stuff changes we'll set state with the new value. The `minimumValue` and `maximumValue` I chose at complete randomness.

## DONE!

We can finally control different properties of shaders that wrap arbitrary images. Go us. Go Shaders. Go OpenGLES 2.X spec.

### As always, live demo on [https://rnplay.org/apps/I9G83g](https://rnplay.org/apps/I9G83g)


{% img http://i.imgur.com/IQDx6Ls.gif Guess the city and win %}



## Full Code

{% raw %}

```
var React = require('react-native');
var GL = require('gl-react-native');

var {
  AppRegistry,
  StyleSheet,
  Text,
  Image,
  Dimensions,
  View,
  ScrollView,
  SliderIOS
} = React;

var {
  width,
  height
} = Dimensions.get('window');




var rn_instagram = React.createClass({
  getInitialState: function() {
    return {
      width:0,
      height: 0,
      saturation: 1,
      brightness: 1,
      contrast: 1,
      hue: 0,
      sepia: 0,
      gray: 0,
      mixFactor: 0
    };
  },
  renderWithDimensions: function(layout) {
    var {
      width,
      height
    } = layout.nativeEvent.layout;
    this.setState({
      width,
      height
    })
  },
  getImage: function() {
    return (
      <Instagram 
        brightness={this.state.brightness}
        saturation={this.state.saturation}
        contrast={this.state.contrast}
        hue={this.state.hue}
        gray={this.state.gray}
        sepia={this.state.sepia}
        mixFactor={this.state.mixFactor}
        width={this.state.width}
        height={this.state.height}
      >
        <Image
          source={{uri: 'http://i.imgur.com/dSIa9jl.jpg'}}
          style={styles.cover}
          resizeMode="cover"
        />
      </Instagram>

    )
  },
  render: function() {
    return (
      <View style={styles.container}>
        <View style={styles.container} onLayout={this.renderWithDimensions}>
          { this.state.width ? this.getImage() : null}
        </View>
        <ScrollView style={styles.container}>
          <View>
            <Text>Blend Factor: {this.state.mixFactor}</Text>
            <SliderIOS
              value={this.state.mixFactor}
              minimumValue={0}
              maximumValue={2}
              onValueChange={(mixFactor) => this.setState({mixFactor})}
            />
          </View>
          <View>
            <Text>Brightness: {this.state.brightness}</Text>
            <SliderIOS
              value={this.state.brightness}
              minimumValue={0}
              maximumValue={3}
              onValueChange={(brightness) => this.setState({brightness})}
            />
          </View>
          <View>
            <Text>Saturation: {this.state.saturation}</Text>
            <SliderIOS
              value={this.state.saturation}
              minimumValue={0}
              maximumValue={3}
              onValueChange={(saturation) => this.setState({saturation})}
            />
          </View>
          <View>
            <Text>Contrast: {this.state.contrast}</Text>
            <SliderIOS
              value={this.state.contrast}
              minimumValue={0}
              maximumValue={3}
              onValueChange={(contrast) => this.setState({contrast})}
            />
          </View>
          <View>
            <Text>Sepia: {this.state.sepia}</Text>
            <SliderIOS
              value={this.state.sepia}
              minimumValue={0}
              maximumValue={1}
              onValueChange={(sepia) => this.setState({sepia})}
            />
          </View>
          <View>
            <Text>Grayscale: {this.state.gray}</Text>
            <SliderIOS
              value={this.state.gray}
              minimumValue={0}
              maximumValue={1}
              onValueChange={(gray) => this.setState({gray})}
            />
          </View>
          <View>
            <Text>Hue: {this.state.hue}</Text>
            <SliderIOS
              value={this.state.hue}
              minimumValue={0}
              maximumValue={10}
              onValueChange={(hue) => this.setState({hue})}
            />
          </View>
        </ScrollView>
      </View>
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1
  },
  cover: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0
  }

});



const shaders = GL.Shaders.create({
  instagram: {
    frag: `
      precision highp float;
      varying vec2 uv;
      uniform sampler2D tex;
      uniform float saturation;
      uniform float brightness;
      uniform float contrast;
      uniform float hue;
      uniform float gray;
      uniform float sepia;
      uniform float mixFactor;

      const vec3 W = vec3(0.2125, 0.7154, 0.0721);
      const mat3 rgb2yiq = mat3(0.299, 0.587, 0.114, 0.595716, -0.274453, -0.321263, 0.211456, -0.522591, 0.311135);
      const mat3 yiq2rgb = mat3(1.0, 0.9563, 0.6210, 1.0, -0.2721, -0.6474, 1.0, -1.1070, 1.7046);
      const vec3 SEPIA = vec3(1.2, 1.0, 0.8);

      vec3 BrightnessContrastSaturation(vec3 color, float brt, float con, float sat)
      {
        vec3 black = vec3(0., 0., 0.);
        vec3 middle = vec3(0.5, 0.5, 0.5);
        float luminance = dot(color, W);
        vec3 gray = vec3(luminance, luminance, luminance);
        
        vec3 brtColor = mix(black, color, brt);
        vec3 conColor = mix(middle, brtColor, con);
        vec3 satColor = mix(gray, conColor, sat);
        return satColor;
      }

      vec3 multiplyBlender(vec3 Color, vec3 filter){
        vec3 filter_result;
        float luminance = dot(filter, W);
        
        if(luminance < 0.5)
          filter_result = 2. * filter * Color;
        else
          filter_result = Color;
            
        return filter_result;
      }

      vec3 ovelayBlender(vec3 Color, vec3 filter){
        vec3 filter_result;

        float luminance = dot(filter, W);
        
        if(luminance < 0.5)
          filter_result = 2. * filter * Color;
        else
          filter_result = 1. - (1. - (2. *(filter - 0.5)))*(1. - Color);
          
        return filter_result;
      }

      vec3 applyHue(vec3 Color, float h) {
        vec3 yColor = rgb2yiq * Color;
        float originalHue = atan(yColor.b, yColor.g);
        float finalHue = originalHue + (h);
        float chroma = sqrt(yColor.b*yColor.b+yColor.g*yColor.g);
        vec3 yFinalColor = vec3(yColor.r, chroma * cos(finalHue), chroma * sin(finalHue));
        return vec3(yiq2rgb*yFinalColor);
      }

      vec3 applyGray(vec3 Color, float g) {
        float gray = dot(Color, vec3(0.299, 0.587, 0.114));
        return mix(Color, vec3(gray, gray, gray), g);
      }

      vec3 applySepia(vec3 Color, float s) {
        float gray = dot(Color, vec3(0.299, 0.587, 0.114));
        return mix(Color, vec3(gray) * SEPIA, s);
      }


      void main() {
        vec2 st = uv.st;
        vec3 irgb = texture2D(tex, st).rgb;
        vec3 filter = texture2D(tex, st).rgb;

        vec3 bcs_result = BrightnessContrastSaturation(irgb, brightness, contrast, saturation);
        vec3 hue_result = applyHue(bcs_result, hue);
        vec3 sepia_result = applySepia(hue_result, sepia);
        vec3 gray_result = applyGray(sepia_result, gray);

        vec3 after_filter = mix(gray_result, multiplyBlender(gray_result, filter), mixFactor);
        
        gl_FragColor = vec4(after_filter, 1.);
      }
    `
  }
});

var Instagram = GL.createComponent(
  ({ brightness, saturation, contrast, hue, gray, sepia, mixFactor, children, ...rest }) =>
  <GL.View
    {...rest}
    shader={shaders.instagram}
    uniforms={{ brightness, saturation, contrast, hue, gray, sepia, mixFactor }}>
    <GL.Uniform name="tex">{children}</GL.Uniform>
  </GL.View>
, { displayName: "Instagram" });


module.exports = rn_instagram;

AppRegistry.registerComponent('rn_instagram', () => rn_instagram);

```
{% endraw %}