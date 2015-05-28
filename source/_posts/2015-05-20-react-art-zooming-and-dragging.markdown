---
layout: post
title: "React Art Zooming and Dragging"
date: 2015-05-20 15:32
comments: true
categories: react react-art zooming dragging d3 svg canvas
---

React-art is awesome, you can easily embody the same concepts in React and your visualizations magically work.

I personally have not done a ton of visualization, and the little I have done is mostly rendering graphs with D3.

I've been tasked with doing a dive for a difficult visualization. We ran into a scenario where we needed the ability to zoom and drag the canvas. D3 conveniently comes with zoom/drag behaviors. D3 integrates pretty well with react-art for doing a lot of the math/generating paths, however after watching [React.js Conf - Scalable Data Visualization](https://www.youtube.com/watch?v=2ii1lEkIv1s) the things immediatley called out that D3 doesn't integrate with react-art are transitions and behaviors (zoom/drag).

So immediately I'm thinking about how to accomplish this. Do I need a global scaler that scales all of my coordinates for zooming? Do I need to manage a coordinate system and adjust all of my coordinates with the dragged X/Y offsets. 

I googled around, and a few people recommended using `canvasEl.getContext('2d').translate(x,y)`. I gave this a try with refs, that didn't work.

It did lead me down the right path though. What if I was able to just utilize one global wrapper, and all of my other code could remain unchanged. The great thing about `Group` is that the coordinate system of the children gets reset, so `0,0` is now the `x,y` of the `Group`

Example:
```
<Group x={100} y={100}>
    <Circle radius={10} stroke="#000" strokeWidth={3} x={20} y={20}/>
</Group>
```

The coordinates of the circle on the whole canvas would actually be `120,120` but because of the group at `x = 100, y = 100` we just need to say `x = 20, y = 20`.


Now that we know that our parent coordinate system effects our child coordinate systems lets prove our final theory that we can have one master parent to control zoom/drag.

Lets start with a base renderer

```
//Assuming React, and react-art are included
var ZoomDragCircle = React.createClass({
    
    render: function() {
        return (
            <Surface
                width={viewportWidth}
                height={viewportHeight}
            >
            </Surface> 
        );
    }

})

```

We have a surface so the next lets get something rendering


```
var ZoomDragCircle = React.createClass({
    
    render: function() {
        return (
            <Surface
                width={viewportWidth}
                height={viewportHeight}
            >
                <Circle x={10} y={10} radius={5} fill="#000" />
            </Surface> 
        );
    }

})

```

Lets add in our drag concept.

```
var ZoomDragCircle = React.createClass({
    getInitialState: function() {
        return {
            x: 0,
            y: 0
        }
    },
    handleMouseDown: function() {
        this.dragging = true;
    },
    handleMouseUp: function() {
        this.dragging = false;
    },
    render: function() {
        return (
            <div 
                    onMouseDown={this.handleMouseDown}
                    onMouseUp={this.handleMouseUp}
            >
                <Surface
                    width={viewportWidth}
                    height={viewportHeight}

                >
                    <Group x={this.state.x} y={this.state.y}>
                        <Circle x={10} y={10} radius={5} fill="#000" />
                    </Group>
                </Surface> 
            </div>
        );
    }

})

```

One thing you'll notice here is the wrapping div. The `react-art` `Surface` element doesn't have the `EventMixin` so it will not register mouse events. We could wrap our `Group` with another `Group` for dragging/zoom however an outer `div` is much easier for now.

You also may notice that we have a slight issue. `onMouseUp` should be globally on the `document` since the `mouseup` event will only be fired if the `mouseup` happens on our wrapping div. For simplicity sake we'll keep it on the div.


So we have a way to toggle whether we are dragging or not, and have the ability to adjust the `x,y` coords of a parent group. Lets actually implement drag.

```
var ZoomDragCircle = React.createClass({
    getInitialState: function() {
        return {
            x: 0,
            y: 0
        }
    },
    componentDidMount: function() {
        document.addEventListener('mousemove', this.handleMouseMove, false);
    },
    componentWillUnmount: function() {
        //Don't forget to unlisten!
        document.removeEventListener('mousemove', this.handleMouseMove, false);
    },
    handleMouseDown: function(e) {
        this.dragging = true;
        //Set coords
          this.coords = {
            x: e.pageX,
            y: e.pageY
          }
    },
    handleMouseUp: function() {
        this.dragging = false;
        this.coords = {};
    },
    handleMouseMove: function(e) {
    //If we are dragging
      if (this.dragging) {
          e.preventDefault();

        //Get mouse change differential
        var xDiff = this.coords.x - e.pageX,
            yDiff = this.coords.y - e.pageY;

        //Update to our new coordinates
            this.coords.x = e.pageX;
            this.coords.y = e.pageY;
        //Adjust our x,y based upon the x/y diff from before
        var x = this.state.x - xDiff,       
            y = this.state.y - yDiff;

        //Re-render
        this.setState(this.state);  
      }

    },
    render: function() {
        return (
            <div 
                    onMouseDown={this.handleMouseDown}
                    onMouseUp={this.handleMouseUp}
            >
                <Surface
                    width={viewportWidth}
                    height={viewportHeight}

                >
                    <Group x={this.state.x} y={this.state.y}>
                        <Circle x={10} y={10} radius={5} fill="#000" />
                    </Group>
                </Surface> 
            </div>
        );
    }
})

```

Now if you spin this up you'll see we can drag around the canvas and our `Circle` will stay the same place. 
Lets do zoom now.

To understand what we're about to do the Art library will translate our `x,y` coords to a `matrix` that is set on the `transform` attribute of the svg `g` element or in the canvas case translated to the appropriate coordinates.

The `matrix` system can be read about here on [MDN](https://developer.mozilla.org/en-US/docs/Web/API/SVGMatrix). Ultimately it allows us to modify the coordinate system (`x,y`) and additionally the scale.

Think of scale as a default multiplier times the size of stuff.

So a scale of `1` means if something is a width of `10` then it would still be 10.
But If we set our scale to `2` and the same width of `10` then `10*2 = 20`. The item would appear larger at 20 pixels.

This is the rough idea behind scale, however we aren't adjusting widths the scale is actually effecting the `x,y` coordinates you are setting. You can define `scaleX` and `scaleY` to be different numbers causing your visual elements to appear blurred/skewed.


```
var ZoomDragCircle = React.createClass({
    getInitialState: function() {
        return {
            x: 0,
            y: 0,
            scale: 1
        }
    },
    componentDidMount: function() {
        document.addEventListener('mousemove', this.handleMouseMove, false);
    },
    componentWillUnmount: function() {
        //Don't forget to unlisten!
        document.removeEventListener('mousemove', this.handleMouseMove, false);
    },
    handleMouseDown: function(e) {
        this.dragging = true;
        //Set coords
          this.coords = {
            x: e.pageX,
            y: e.pageY
          }
    },
    handleMouseUp: function() {
        this.dragging = false;
        this.coords = {};
    },
    handleMouseMove: function(e) {
    //If we are dragging
      if (!this.dragging) {
        return;
      }
          e.preventDefault();

        //Get mouse change differential
        var xDiff = this.coords.x - e.pageX,
            yDiff = this.coords.y - e.pageY;

        //Update to our new coordinates
            this.coords.x = e.pageX;
            this.coords.y = e.pageY;
        //Adjust our x,y based upon the x/y diff from before
        var x = this.state.x - xDiff,       
            y = this.state.y - yDiff;

        //Re-render
        this.setState(this.state);  

    },
    //So we can handle the mousewheel returning -0 or 0
    isNegative: function (n) {
      return ((n = +n) || 1 / n) < 0;
    },
    handleMouseWheel: function(e) {
      var ZOOM_STEP = .03;

        //require the shift key to be pressed to scroll
        if (!e.shiftKey) {
            return;
        }
      e.preventDefault();
      var direction = (this.isNegative(e.deltaX) &&  this.isNegative(e.deltaY) ) ? 'down' : 'up';

      if (direction == 'up') {
        this.state.scale += ZOOM_STEP;
      } else {
        this.state.scale -= ZOOM_STEP;
      }

      this.state.scale = this.state.scale < 0 ? 0 : this.state.scale;

      this.setState(this.state);
    },
    render: function() {
        return (
            <div 
                onMouseDown={this.handleMouseDown}
                onMouseUp={this.handleMouseUp}
                onWheel={this.handleMouseWheel}
            >
                <Surface
                    width={viewportWidth}
                    height={viewportHeight}

                >
                    <Group 
                        x={this.state.x} 
                        y={this.state.y}
                        scaleX={this.state.scale}
                        scaleY={this.state.scale}
                    >
                        <Circle x={10} y={10} radius={5} fill="#000" />
                    </Group>
                </Surface> 
            </div>
        );
    }

})

```

Full working demo, hold shift and use your mouse wheel/track pad to zoom or just grab and drag around.

<p data-height="624" data-theme-id="0" data-slug-hash="jPMMao" data-default-tab="result" data-user="browniefed" class='codepen'>See the Pen <a href='http://codepen.io/browniefed/pen/jPMMao/'>jPMMao</a> by Jason Brown (<a href='http://codepen.io/browniefed'>@browniefed</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

Now we should be able to zoom in/zoom out while holding shift key + using your scroll wheel.
If you want a predictable scale you can add some `+` and `-` buttons somwhere and just increment `this.state.scale`

I'm hoping to do more write ups and examples with react-art. The great thing is that you can render react-art with react-native. With appropriate abstractions you could possibly have the same visualizations on the web as you do on native.


