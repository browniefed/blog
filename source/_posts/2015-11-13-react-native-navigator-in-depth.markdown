---
layout: post
title: "React Native - Navigator in Depth"
date: 2015-11-13 19:36
comments: true
categories: react-native navigator
published: false
---

If you haven't read the post by [James Ide](https://twitter.com/JI) on how they solved the most common problems with navigating in React Native with ExNavigator then you should go read it here [https://medium.com/the-exponent-log/routing-and-navigation-in-react-native-6b27bee39603](https://medium.com/the-exponent-log/routing-and-navigation-in-react-native-6b27bee39603). No really go read it. We'll cover some of the topics, but we'll solve them with the normal `Navigator`.

Another great post from the [Brian Leonard](https://twitter.com/bleonard) at TaskRabbit explaining a lot of Navigation in React Native applications, read here [http://tech.taskrabbit.com/blog/2015/09/21/react-native-example-app/](http://tech.taskrabbit.com/blog/2015/09/21/react-native-example-app/).

FYI they're hiring!

## Navigator Core Concepts

* It's all components! Remember, React is all about components, and Navigator is no different.
* Routes are just objects, `Navigator` doesn't care what you put on them.
* `Navigator` should be at the root of your application, or as close as possible to the root, you may need to do some modals.
* DO NOT USE `NavigatorIOS` unless you know what you're doing. Stick with `Navigator` you'll be happier in the long run.


## Navigator Basics

Alright lets get into some basics.
We'll start with just an empty `Navigator`.
Lets pretend we're in our `index.ios.js` or `index.android.js`, some other top level area.

```
render: function() {
	return (
		<Navigator

		/>
	)	
}

```

Hey we have a worthless `Navigator` lets make it do something.

```
renderScene: function(route, navigator) {
	return <View><Text>Hello World</Text></View>
},

render: function() {
	return (
		<Navigator
			renderScene={this.renderScene}
		/>
	)	
}

```

Okay a little better, still won't render because didn't specify an `initialRoute`

```
renderScene: function(route, navigator) {
	return <View><Text>{route.text}</Text></View>
},

render: function() {
	return (
		<Navigator
			initialRoute={{text: 'Hello World'}}
			renderScene={this.renderScene}
		/>
	)	
}

```
In theory this should working! We define an `initialRoute`, that is just an arbitrary object we created. We happened to put the `text` we want to render on it, but think bigger.
We can put components to render, props to pass, in the case of `NavigationBar` we can put dynamic titles, dyanmic buttons, etc. The possibilities are endless.

Now to go to a different route you have to options, you can get access to the `navigator` and pass that down to each component in `renderScene` or additionall you can just get a `ref` to it at this top level.

```
whenSomethingIsPressed: function() {
	this.refs.navigator.push({
		text: 'Hey we went to a new screen!'
	})
},

renderScene: function(route, navigator) {
	return <View><Text>{route.text}</Text></View>
},

render: function() {
	return (
		<Navigator
			ref="navigator"
			initialRoute={{text: 'Hello World'}}
			renderScene={this.renderScene}
		/>
	)	
}

```

Yes nothing is calling `whenSomethingIsPressed` yet but as you can see here, it gets access to the navigator we have. Pushes a new route, and thus we will go through our cycle of animating from the current screen to a new screen.

That looks like this.

{% img http://i.imgur.com/824euyt.gif that is pretty boring %}


## Navigator Internals

You may not have realized it but `Navigator` uses `Immutable` heavily. The entire route stack is immutable, which is why every time you pop back to a route you're right back exactly how you left it.