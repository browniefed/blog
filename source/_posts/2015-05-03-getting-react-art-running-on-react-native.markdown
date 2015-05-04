---
layout: post
title: "Getting react-art running on react-native"
date: 2015-05-03 20:33
comments: true
categories: react-art react-native react
---

I hadn't realized that the react-art native rendering stuff had landed in into the react-native core.
So I figured I'd try and get in running. I am an XCode noob so I'll be sure and be descriptive.

First off lets set up a react-native project. (Assuming you have react-native cli installed);

`react-native init react_art_demo`

The current released version of `react-native` doesn't have the ART libs in it so in our `node_modules` folder we can delete the `react-native` in there and do a git clone of master `git clone https://github.com/facebook/react-native.git`

We will also need the `art` lib itself installed. So `cd` to the root of `react_art_demo`.
Then run `npm install art`


Once we have the master repo of `react-native`, and `art` installed we can open up our `react_art_demo` project in XCode.

We will need to do 2 things.

* Add the `ART.xcodeproj` to XCode
* Link the binary


To add ART.xcodeproj find the file located in `react-native/Libraries/ART/ART.xcodeproj` and just drag it over to the `Libraries` section in XCode.
That should look like so

{% img http://i.imgur.com/TWeDUTa.png Add ART.xcodeproj to Libraries %}

Next we'll link the binary.

With the root `react_art_demo` project selected on the left, select `Build Phases` from the center view.
There will be a section called "Link Binary With Libraries", expand it, press the + and select `libART.a`

Like so

{% img http://i.imgur.com/lIMZkfh.png Link binary%}

Just as a warning, I didn't link the binary and got a few "No manager class found for view with module name" errors.


Now go forth and write your `react-art`. I always like to get the `vector-widget` example running so to do that here is the code.

`index.ios.js`

```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 */
'use strict';

var React = require('react-native');
var {
  AppRegistry,
  StyleSheet,
  Text,
  View,
} = React;

var VectorWidget = require('./VectorWidget');

var react_art_demo = React.createClass({

  render: function() {
    return (
      <View style={styles.container}>
        <VectorWidget style={styles.vector}/>
      </View>
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    backgroundColor: '#FFF',
  },
  vector: {
    width: 100,
    height: 100
  },
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
  instructions: {
    color: '#333333',
    marginBottom: 5,
    marginTop: 100
  },
});

AppRegistry.registerComponent('react_art_demo', () => react_art_demo);
```

And your `VectorWidget.js` file

```
/**
 * Copyright 2013 Facebook, Inc.
 * All rights reserved.
 *
 * This source code is licensed under the BSD-style license found in the
 * LICENSE file in the root directory of this source tree. An additional grant
 * of patent rights can be found in the PATENTS file in the same directory.
 */
"use strict";

var React = require('react-native');
var ReactArt = require('./node_modules/react-native/Libraries/ART/ReactIOSART');
var {
    Art,
    TouchableWithoutFeedback
} = React;

var {
    Group,
    Shape,
    Surface,
    Transform
} = ReactArt;

var MOUSE_UP_DRAG = 0.978;
var MOUSE_DOWN_DRAG = 0.9;
var MAX_VEL = 11;
var CLICK_ACCEL = 3;
var BASE_VEL = 0.15;

/**
 * An animated SVG component.
 */
var VectorWidget = React.createClass({
  /**
   * Initialize state members.
   */
  getInitialState: function() {
    return {degrees: 0, velocity: 0, drag: MOUSE_UP_DRAG};
  },

  /**
   * When the component is mounted into the document - this is similar to a
   * constructor, but invoked when the instance is actually mounted into the
   * document. Here's, we'll just set up an animation loop that invokes our
   * method. Binding of `this.onTick` is not needed because all React methods
   * are automatically bound before being mounted.
   */
  componentDidMount: function() {
    this._interval = window.setInterval(this.onTick, 20);
  },

  componentWillUnmount: function() {
    window.clearInterval(this._interval);
  },

  onTick: function() {
    var nextDegrees = this.state.degrees + BASE_VEL + this.state.velocity;
    var nextVelocity = this.state.velocity * this.state.drag;
    this.setState({degrees: nextDegrees, velocity: nextVelocity});
  },

  /**
   * This is the "main" method for any component. The React API allows you to
   * describe the structure of your UI component at *any* point in time.
   */
  render: function() {
    return (
      <Surface
        width={700}
        height={700}
        {this.renderGraphic(this.state.degrees)}
      </Surface>
    );
  },

  /**
   * Better SVG support for React coming soon.
   */
  renderGraphic: function(rotation) {

    return (
      <Group
      >
        <Group x={210} y={135}>
          <Shape fill="rgba(0,0,0,0.1)" d={BORDER_PATH} />
          <Shape fill="#7BC7BA" d={BG_PATH} />
          <Shape fill="#DCDCDC" d={BAR_PATH} />
          <Shape fill="#D97B76" d={RED_DOT_PATH} />
          <Shape fill="#DBBB79" d={YELLOW_DOT_PATH} />
          <Shape fill="#A6BD8A" d={GREEN_DOT_PATH} />
          <Group x={55} y={29}>
            <Group rotation={rotation} originX={84} originY={89}>
              <Shape fill="#FFFFFF" d={CENTER_DOT_PATH} />
              <Group>
                <Shape d={RING_ONE_PATH} stroke="#FFFFFF" strokeWidth={8} />
                <Shape d={RING_TWO_PATH} transform={RING_TWO_ROTATE} stroke="#FFFFFF" strokeWidth={8} />
                <Shape d={RING_THREE_PATH} transform={RING_THREE_ROTATE} stroke="#FFFFFF" strokeWidth={8} />
              </Group>
            </Group>
          </Group>
        </Group>
      </Group>
    );
  }
});

var BORDER_PATH = "M3.00191459,4 C1.34400294,4 0,5.34785514 0,7.00550479 L0,220.994495 C0,222.65439 1.34239483,224 3.00191459,224 L276.998085,224 C278.655997,224 280,222.652145 280,220.994495 L280,7.00550479 C280,5.34561033 278.657605,4 276.998085,4 L3.00191459,4 Z M3.00191459,4";
var BG_PATH = "M3.00191459,1 C1.34400294,1 0,2.34785514 0,4.00550479 L0,217.994495 C0,219.65439 1.34239483,221 3.00191459,221 L276.998085,221 C278.655997,221 280,219.652145 280,217.994495 L280,4.00550479 C280,2.34561033 278.657605,1 276.998085,1 L3.00191459,1 Z M3.00191459,1";
var BAR_PATH = "M3.00191459,0 C1.34400294,0 0,1.34559019 0,3.00878799 L0,21 C0,21 0,21 0,21 L280,21 C280,21 280,21 280,21 L280,3.00878799 C280,1.34708027 278.657605,0 276.998085,0 L3.00191459,0 Z M3.00191459,0";
var RED_DOT_PATH = "M12.5,17 C16.0898511,17 19,14.0898511 19,10.5 C19,6.91014895 16.0898511,4 12.5,4 C8.91014895,4 6,6.91014895 6,10.5 C6,14.0898511 8.91014895,17 12.5,17 Z M12.5,17";
var YELLOW_DOT_PATH = "M31.5,17 C35.0898511,17 38,14.0898511 38,10.5 C38,6.91014895 35.0898511,4 31.5,4 C27.9101489,4 25,6.91014895 25,10.5 C25,14.0898511 27.9101489,17 31.5,17 Z M31.5,17";
var GREEN_DOT_PATH = "M50.5,17 C54.0898511,17 57,14.0898511 57,10.5 C57,6.91014895 54.0898511,4 50.5,4 C46.9101489,4 44,6.91014895 44,10.5 C44,14.0898511 46.9101489,17 50.5,17 Z M50.5,17";
var CENTER_DOT_PATH = "M84,105 C92.8365564,105 100,97.8365564 100,89 C100,80.1634436 92.8365564,73 84,73 C75.1634436,73 68,80.1634436 68,89 C68,97.8365564 75.1634436,105 84,105 Z M84,105";
var RING_ONE_PATH = "M84,121 C130.391921,121 168,106.673113 168,89 C168,71.3268871 130.391921,57 84,57 C37.6080787,57 0,71.3268871 0,89 C0,106.673113 37.6080787,121 84,121 Z M84,121";
var RING_TWO_PATH = "M84,121 C130.391921,121 168,106.673113 168,89 C168,71.3268871 130.391921,57 84,57 C37.6080787,57 0,71.3268871 0,89 C0,106.673113 37.6080787,121 84,121 Z M84,121";
var RING_THREE_PATH = "M84,121 C130.391921,121 168,106.673113 168,89 C168,71.3268871 130.391921,57 84,57 C37.6080787,57 0,71.3268871 0,89 C0,106.673113 37.6080787,121 84,121 Z M84,121";
var RING_TWO_ROTATE = new Transform().translate(84.000000, 89.000000).rotate(-240.000000).translate(-84.000000, -89.000000);
var RING_THREE_ROTATE = new Transform().translate(84.000000, 89.000000).rotate(-300.000000).translate(-84.000000, -89.000000);

module.exports = VectorWidget;
```


You may notice that I removed the `mouseDown/mouseUp` stuff.
I tried getting it work very briefly with `onPress` and `TouchableWithoutFeedback` but I couldn't get it working.
If you get it figured out let me know.

##Conclusion
That's it, you can now render your react-art code natively. Enjoy

{% img http://i.imgur.com/AXbsaTB.png Final result %}