---
layout: post
title: "React Native - Easy Overlay Modal with Navigator"
date: 2015-10-18 01:43
comments: true
categories: react-native react modal navigator overlay
---

It pays to have the `Navigator` at the root of your application. This allows you to tunnel back and render something at the root. In our case a custom Modal overlay component. You can pass anything on the route object, and anytime you render the same component at the same place it will just re-render that same component. So lets use the power of React to solve our problems.

## What are we making

{% img http://i.imgur.com/5LhPD3l.gif I should have made this loop %}

<!-- more -->

## Setup

Lets setup our app

```
var React = require('react-native');
var {
  AppRegistry,
  StyleSheet,
  Text,
  View,
  Navigator,
  TouchableOpacity,
  Animated,
  Dimensions
} = React;

var {
  height: deviceHeight
} = Dimensions.get('window');
```

We'll get our `deviceHeight` so we can manually animate our modal up. You could use `LayoutAnimation` here as well and not deal with getting the deviceHeight but I like `Animated` so deal with it.


## Route Stack

```
var RouteStack = {
    app: {
      component: App 
    }
}
```

This is our super complex route stack. All we do is have a named route with the componet to render.

## Root Application
```
var ModalApp = React.createClass({
  getInitialState: function() {
    return { modal: false };
  },
  renderScene: function(route, navigator) {
    var Component = route.component;
    
    return (
      <Component openModal={() => this.setState({modal: true})}/>
    );
  },
  render: function() {
    return (
      <View style={styles.container}>
        <Navigator
            initialRoute={RouteStack.app}
            renderScene={this.renderScene}
        />
        {this.state.modal ? <TopModal closeModal={() => this.setState({modal: false}) }/> : null }
      </View>
    );
  }
});

```

This is the root of our application. Our render function is pretty basic. We render `Navigator` with our `intialRoute` being our only Route. Our `renderScene` function is going to control our logic. 

When we render our component we pass down an `openModal` function. This will set `modal:true` on our state which will allow for us to open/close the modal over the current route. This will just cause `Navigator` to re-render at the current route. This means your rendered `Component` at the current route will have `componentWillReceiveProps` triggered. Our `TopModal` will receive a `closeModal` function to set `modal:false` on state and unmount our `TopModal`.

We put our modal after the `Navigator` so we can render on top of it.

## Open Modal

```
var App = React.createClass({
    render: function() {
      return (
        <View style={styles.flexCenter}>
          <TouchableOpacity onPress={this.props.openModal}>
            <Text>Open Modal</Text>  
          </TouchableOpacity>
        </View>
      )
    }
});

```

All we do is when we want to open the modal just call the `openModal` function on props. That will call up to the function in `Navigator` `renderScene` and pop open the modal over the existing app.

## The Modal
{% raw %}
```
var TopModal = React.createClass({
  getInitialState: function() {
    return { offset: new Animated.Value(deviceHeight) }
  },
  componentDidMount: function() {
    Animated.timing(this.state.offset, {
      duration: 100,
      toValue: 0
    }).start();
  },
  closeModal: function() {
    Animated.timing(this.state.offset, {
      duration: 100,
      toValue: deviceHeight
    }).start(this.props.closeModal);
  },
  render: function() {
    return (
        <Animated.View style={[styles.modal, styles.flexCenter, {transform: [{translateY: this.state.offset}]}]}>
          <TouchableOpacity onPress={this.closeModal}>
            <Text style={{color: '#FFF'}}>Close Menu</Text>
          </TouchableOpacity>
        </Animated.View>
    )
  }
});
```
{% endraw %}
Here we have a basic modal. We set the `translateY` to the full device height so that it renders off screen, and on mount we slide it up in `100ms`. On close we slide it down, call the `closeModal` which will trigger the re-render in our `renderScene`. This case we won't have `modal: true` set so our `TopModal` will just unmount.


## Done

Hey now go get your modal on. Just remember, React is flexible. Sometimes you need to pass something up to render at the top. Yes slightly a pain, but it's a manageable pain.

#### Live Demo https://rnplay.org/apps/kF7avw

{% img http://i.imgur.com/5LhPD3l.gif I should have made this loop %}


## Full Code

{% raw %}
```
var React = require('react-native');
var {
  AppRegistry,
  StyleSheet,
  Text,
  View,
  Navigator,
  TouchableOpacity,
  Animated,
  Dimensions
} = React;

var {
  height: deviceHeight
} = Dimensions.get('window');

var TopModal = React.createClass({
  getInitialState: function() {
    return { offset: new Animated.Value(deviceHeight) }
  },
  componentDidMount: function() {
    Animated.timing(this.state.offset, {
      duration: 100,
      toValue: 0
    }).start();
  },
  closeModal: function() {
    Animated.timing(this.state.offset, {
      duration: 100,
      toValue: deviceHeight
    }).start(this.props.closeModal);
  },
  render: function() {
    return (
        <Animated.View style={[styles.modal, styles.flexCenter, {transform: [{translateY: this.state.offset}]}]}>
          <TouchableOpacity onPress={this.closeModal}>
            <Text style={{color: '#FFF'}}>Close Menu</Text>
          </TouchableOpacity>
        </Animated.View>
    )
  }
});

var App = React.createClass({
    render: function() {
      return (
        <View style={styles.flexCenter}>
          <TouchableOpacity onPress={this.props.openModal}>
            <Text>Open Modal</Text>  
          </TouchableOpacity>
        </View>
      )
    }
});

var RouteStack = {
  app: {
    component: App 
  }
}

var ModalApp = React.createClass({
  getInitialState: function() {
    return {
      modal: false 
    };
  },
  renderScene: function(route, navigator) {
    var Component = route.component;
    return (
      <Component openModal={() => this.setState({modal: true})}/>
    )
  },
  render: function() {
    return (
      <View style={styles.container}>
        <Navigator
          initialRoute={RouteStack.app}
          renderScene={this.renderScene}
        />
        {this.state.modal ? <TopModal closeModal={() => this.setState({modal: false}) }/> : null }
      </View>
    );
  }
});



var styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  flexCenter: {
    flex: 1,
    justifyContent: 'center', 
    alignItems: 'center'
  },
  modal: {
    backgroundColor: 'rgba(0,0,0,.8)',
    position: 'absolute',
    top: 0,
    right: 0,
    bottom: 0,
    left: 0
  }
});

```

{% endraw %}
