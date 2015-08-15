---
layout: post
title: "CSS-Layout and React-Art"
date: 2015-06-01 10:29
comments: true
categories: css-layout react-native react react-art visualization layout
---

### Intro

If you've used `react-native` before then you may know that "css" you're writing isn't actually CSS. It's a descriptor for a layout engine. That layout engine is [css-layout](https://github.com/facebook/css-layout) which was created by [@vjeux](https://twitter.com/Vjeux) and compiled to Java/Objective-C.

Layout is difficult. There have been attempts at various constraint based layouts in JavaScript, [GSS](http://gridstylesheets.org/) is one of those. Which is a port of Apples `Cassowary` constraint solver which was also ported to JavaScript [https://github.com/slightlyoff/cassowary.js](https://github.com/slightlyoff/cassowary.js).

Now this is all fine and well but from the bit I've read constraints fall down sometimes. This usually happens when you don't specify enough constraints based on the current layout.


### Layout is hard!

Parent layouts get effected by their child layouts. CSS is weird in that you can remove items from layout with `position: absolute` but ultimately the top parent `width/height` is directly effected by it's children.

This is an over simplification but many times in `React` you have to hook into `componentDidMount` get the child width and take action.

An example would be even width labels in a form. We could measure the text but depending on `font`, `font-size` it could measure each value differently. So we hide the form on initial render, get the max label width and set it on state.

<!-- more -->


### CSS-Layout Basics

It takes a subset of flexbox and some other styling parameters and returns `width`, and the `left/top` offsets for each item and it's children.

This example is taken directly from the `css-layout` git repo.

```
computeLayout(
  {style: {padding: 50}, children: [
    {style: {padding: 10, alignSelf: 'stretch'}}
  ]}
);
// =>
{
    width: 120,
    height: 120,
    top: 0,
    left: 0,
    children: [{
        width: 20,
        height: 20,
        top: 50,
        left: 50
    }]
}
```

So explaining this. We have an item with a padding of `50`, so `50px` around the entire item.

It's child item has a padding of `10`, so `10px` all the way around. 

Therefore the parent has `50*2 = 100` initial width/height. Now we take into account the children.

Width/Height: 

```
Parent: `50*2 = 100` +  `10*2 = 20` = `100 + 120`
Child: `10*2 = 20`
```

Position:

```
Parent: `x = 0, y = 0` since we are starting there.
Child: Parent padding `50` so our child is inset at `x = 50, y = 50`
```

Lets change the child width and see what happens
```
computeLayout(
    {
        style: {
            padding: 50
        },
        children: [{
            style: {
                padding: 10,
                width: 1000,
                alignSelf: 'stretch'
            }
        }]
    }
);
// =>
{
    "width": 1100,
    "height": 120,
    "top": 0,
    "left": 0,
    "children": [{
        "width": 1000,
        "height": 20,
        "top": 50,
        "left": 50
    }]
}
```
Because our child defined a width of `1000` we then add on our `50*2` of padding on both sides and now the parent has a width of `1100`.


There are some other nuances that you can read about in the `css-layout` repo.

### Basic Example

First off we'll need a component tree. Now in React they transpile JSX, and build the component tree for us. However in our case we can just create a JSON tree.

```
var componentTree = {
    style: {
        padding: 10
    },
    component: Rectangle,
    children: [{
        style: {
            padding: 10,
            flexDirection: 'column',
            alignItems: 'center'
        },
        component: Rectangle,
        children: [
            {
                style: {
                    width: 30,
                    height: 30
                },
                component: Rectangle
            },
            {
                style: {
                    margin: 10,
                    width: 50,
                    height: 50,
                    alignItems: 'center',
                    justifyContent: 'center'
                },
                component: Circle,
                children: [
                    {
                        style: {
                            width: 10,
                            height: 10
                        },
                        component: Circle
                    }
                ]
            }
        ]
    }]
};

```

We add a `component` parameter to the tree. This is the thing that will be rendered. 
We could add additional properties here, maybe create custom renderers but we'll keep it simple.

The break down is like so
    
- A surrounding rectangle w/ `10px` of padding all around 
- 1 child that is a rectangle with another `10px` of padding, and it's children in a `column` based layout.    
- We align each of the items in the `center`
- 2 children one a Rectangle, one a Cirlce.
- Rectangle = `width = 30, height = 30`
- Circle = `width = 50, height = 50` and a surrounding `10px` margin and it's children centered vertically and horizontally
- That cirlce has a circle inside that is `width = 10` and `height = 10
We'll have to do some basic math on the Circle to compute the radius, and center it correctly.


Now we'll need to traverse the layout.
We'll do that with a function that calls itself

```
function traverseLayout(componentTree, layout) {
    var Component = componentTree.component;
    return (
        <Group x={layout.left} y={layout.top}>
            <Component
                {...getProps(Component, layout)}
             />
             <Group x={0} y={0}>
                {
                    !componentTree.children ? null : componentTree.children.map(function(child, index) {
                        return traverseLayout(child, layout.children[index]);
                    })

                }
             </Group>
        </Group>
    )
}
```
This is a super crude layout renderer but it works for our purposes.

It creates a group and we pass in our `top/left` to `x/y` of the group. This is necessary to make our children `top/left` work correctly.
Then renders the component with selected props. We'll just render `stroke="#000"` and a stroke={1}.

Then if we have children we will call ourself with the child component and layout.

To process the props we need to render different props for specific components.

`Rectangle` needs `width/height` which we have.
`Circle` needs the radius computed, and to then be ofset by the radius. So we just divide the `width/2` and for positioning `y` we divide the `height/2`.

```
function getProps(component, layout) {

    var props = {
        x: 0,
        y: 0
    };

    if (Rectangle === component) {
        props.width = layout.width;
        props.height = layout.height;

    } else if (Circle === component) {
        props.radius = layout.width/2;
        props.x += (layout.width/2)
        props.y += (layout.height/2);
    }

    props.stroke = "#000"; // Just to visualize
    props.strokeWidth = 1; // Just to visualize
    return props;
}
```

That is it, now we can render a tree of `Rectangles` and `Circles`. The complete code is below.

```
var React = require('react');
var ReactArt = require('react-art'),
    Surface = ReactArt.Surface,
    Group = ReactArt.Group,
    computeLayout = require('css-layout'),
    Circle = require('react-art/shapes/circle'),
    Rectangle = require('react-art/shapes/rectangle');

var Surface = ReactArt.Surface;

var viewportWidth = function() {
    return  window.innerWidth - 100;
}
var viewportHeight = function() {
    return window.innerHeight - 100;
}

var componentTree = {
    style: {
        padding: 10
    },
    component: Rectangle,
    children: [{
        style: {
            padding: 10,
            flexDirection: 'column',
            alignItems: 'center'
        },
        component: Rectangle,
        children: [
            {
                style: {
                    width: 30,
                    height: 30
                },
                component: Rectangle
            },
            {
                style: {
                    margin: 10,
                    width: 50,
                    height: 50,
                    alignItems: 'center',
                    justifyContent: 'center'
                },
                component: Circle,
                children: [
                    {
                        style: {
                            width: 10,
                            height: 10
                        },
                        component: Circle
                    }
                ]
            }
        ]
    }]
};

var layout = computeLayout(componentTree);

function traverseLayout(componentTree, layout) {
    var Component = componentTree.component;
    return (
        <Group x={layout.left} y={layout.top}>
            <Component
                {...getProps(Component, layout)}
             />
             <Group x={0} y={0}>
                {
                    !componentTree.children ? null : componentTree.children.map(function(child, index) {
                        return traverseLayout(child, layout.children[index]);
                    })

                }
             </Group>
        </Group>
    )
}

function getProps(component, layout) {

    var props = {
        x: 0,
        y: 0
    };

    if (Rectangle === component) {
        props.width = layout.width;
        props.height = layout.height;

    } else if (Circle === component) {
        props.radius = layout.width/2;
        props.x += (layout.width/2)
        props.y += (layout.height/2);
    }

    props.stroke = "#000";
    props.strokeWidth = 1;
    return props;
}


var Demo = React.createClass({
  getInitialState: function() {
    return {

    }
  },
  getRenderLayout: function() {
    return traverseLayout(componentTree, layout);
  },
  render: function () {
    return (
      <div>
        <Surface
            width={viewportWidth()}
            height={viewportHeight()}
        >
            {this.getRenderLayout()}
        </Surface>
      </div>
    );
  }
});

module.exports = Demo;
```

### Possibilities
Tic-tac-toe? Heh.

This could be made to layout components arbitrarily much like we do with `react-native`. It's not perfect and could only be used in specific scenarios but it's still a fun prototype.

### Demo

<p data-height="624" data-theme-id="0" data-slug-hash="zGZOMN" data-default-tab="result" data-user="browniefed" class='codepen'>See the Pen <a href='http://codepen.io/browniefed/pen/zGZOMN/'>zGZOMN</a> by Jason Brown (<a href='http://codepen.io/browniefed'>@browniefed</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>






