---
layout: post
title: "Create a map with React-Art"
date: 2015-05-30 10:15
comments: true
categories: react-art mapping tiles react react-native
---


### Intro

Just Like the title states, we're going to make a map with `react-art`. When you think of maps many people jump straight to `leaflet`, `google maps`, or `mapbox`. Well one of the key things they are doing is just plotting map tiles. 

Map tiles are just images that can be stitched together and form a map. That is why whenever you drag on a map portions of it pop in in squares.

Don't worry, I won't get deep into mapping terminology because I don't know it. If you want to learn checkout this post [http://www.macwright.org/2012/05/15/how-web-maps-work.html](http://www.macwright.org/2012/05/15/how-web-maps-work.html)

All you'll need to know is `Latitude`, `Longitude`, and `Tile` aka (an image of a piece of a map).

Our tilemap source will be the fabulous [OpenStreetMap](https://www.openstreetmap.org/). It's a community driven mapping solution. Check it out and contribute if you can!

This was going to be a blog post about creating a map but I ended up turning it into a library.

### What I Built

I ended up writing up a library to show it off.

Checkout [https://github.com/browniefed/react-art-map](https://github.com/browniefed/react-art-map) for the library and examples.


We use [https://github.com/zacbarton/node-googlemaps-utils](https://github.com/zacbarton/node-googlemaps-utils) and
[https://github.com/gagan-bansal/map-the-tiles](https://github.com/gagan-bansal/map-the-tiles).


These 2 libraries are helper libraries.

`googlemaps-utils` takes a `width/height` and a central `lat/long` and gives us a bounding box which is just the `top/left` coordinate and the `bottom/right` coordinate.

We then take that bounding box and convert it to meter points so we can generate a [mercator projection](http://en.wikipedia.org/wiki/Mercator_projection).

The `map-the-tiles` takes those meter points and returns `x,y,z` points;

Those `x,y,z` points get fed into the OSM url `http://{s}.tile.osm.org/{z}/{x}/{y}.png` and we get our tile images.

We can then render them at their given `top/left` points w/ `react-art`.

Ultimately this library was built with A LOT of googling and assembling some tools people already constructed. 


### Some Internals

Most of the logic is just in the `TileUtil`. There are likely much more efficient ways to do this however this is my first stab at it with my limited geo knowledge.

Converts `lat/long` to meter points
```
    degrees2meters(lon,lat) {
        var x = lon * 20037508.34 / 180;
        var y = Math.log(Math.tan((90 + lat) * Math.PI / 360)) / (Math.PI / 180);
        y = y * 20037508.34 / 180;
        return [x, y]
    }

``` 

Converts meters to coordinates
```
    meters2degress(x,y) {
        var lon = x *  180 / 20037508.34 ;
        var lat = Number(180 / Math.PI * (2 * Math.atan(Math.exp(y * Math.PI / 180)) - Math.PI / 2));
        return [lon, lat]
    }
```


The main layout generator.

```
    getTileLayout(options) {
        var layout = [];
        var bounds = gmu.calcBounds(options.center[1], options.center[0], options.zoom, options.width, options.height); // GET COORDINATE BOUNDS

        var topLeftMeters = TileUtil.degrees2meters(bounds.left, bounds.top),
            bottomRightMeters = TileUtil.degrees2meters(bounds.right, bounds.bottom);

        //Conver the coordinates each to meters
        
        var tiler = new MapTheTiles(null, options.tileWidth); // Create a generic tiler based on our tile width
        
        var layoutForBounds = {
            top: topLeftMeters[1],
            left: topLeftMeters[0],
            right: bottomRightMeters[0],
            bottom: bottomRightMeters[1]
        };
        //Pass in the meters for each point

        var tiles = tiler.getTiles(layoutForBounds, options.zoom) // Get the x,y,z points for our zoom level

        tiles.forEach(function(tile) {
            var coordPoint = {
                x: tile.X,
                y: tile.Y,
                z: tile.Z
            },
            coord = {
                x: tile.left,
                y: tile.top,
                img: TileUtil.getTileUrl(options.tileSource, coordPoint, options.subdomains) //Just format the OSM tile resource
            };

            layout.push(coord);
        }, this);

        return layout;
    }
```


This is how we render each tile.
We have the `x/y` coordinates thanks to our tiler.

We use `Paths.js` to create a generic rectangular path.
This is so we can support `react-native` in the future since the shapes have yet to be created.

Then we create a new fill with the tile image and set it to the width/height of the generic tile at `0,0` of the shape.

This technically is a pattern for the background but because we set it to the exact `width/height` of the image it just renders the image once.

```
var rectanglePath = Rectangle({
  top: 0,
  left: 0,
  right: 256,
  bottom: 256
}).path.print();
///UP ABOVE

        return layout.map(function(tile) {
            return (
                <Shape
                    d={rectanglePath}
                    x={tile.x}
                    y={tile.y}
                    fill={new Pattern(tile.img, this.props.tileWidth , this.props.tileWidth, 0, 0)}
                />
            )
        }, this);
```


### The Code


```
var React = require('react');
var ReactMap = require('react-art-map');
var ReactArt = require('react-art'),
    Circle = require('react-art/shapes/circle');

var Map = ReactMap.Map;

var viewportWidth = function() {
    return  window.innerWidth - 100;
}
var viewportHeight = function() {
    return window.innerHeight - 100;
}

var center = [
    -122.668197,45.525292
],
offset = 3;

var Demo = React.createClass({
  getInitialState: function() {
    return {
      center: center,
      zoom: 15,
      x: 100
    }
  },
  componentDidMount: function() {
    requestAnimationFrame(this.updateCircle);
  },
  updateCircle: function() {
    if (this.state.x >= viewportWidth()) {
        offset = -3;
    } else if ( this.state.x <= 99) {
        offset = 3;
    }

    this.state.x += offset;
    this.setState(this.state, function() {
        requestAnimationFrame(this.updateCircle);
    });
  },
  handleDrag: function(newCenter) {
    this.setState({
      center: newCenter
    });
  }, 
  render: function () {
    return (
      <div>
        <Map
            width={viewportWidth()}
            height={viewportHeight()}
            center={this.state.center}
            zoom={this.state.zoom}
            tileSource="http://{s}.tile.osm.org/{z}/{x}/{y}.png"
            onDrag={this.handleDrag}
        >
            <Circle 
                x={this.state.x}
                y={100}
                radius={30}
                stroke="#000"
                strokeWidth={5}
            />
        </Map>
      </div>
    );
  }
});

module.exports = Demo;
```



### React Native?!?!?!

React Native has a map implementation but it doesn't allow for much flexibility. You can render pins but that is about it.

With this library once the Pattern fill gets implemented you can render any map tile based service + any cool visualiztions on the map that you want.

I've logged an issue here [https://github.com/facebook/react-native/issues/1462](https://github.com/facebook/react-native/issues/1462) so follow along for when it gets implemented.



### Results

<p data-height="624" data-theme-id="0" data-slug-hash="PqWRvz" data-default-tab="result" data-user="browniefed" class='codepen'>See the Pen <a href='http://codepen.io/browniefed/pen/PqWRvz/'>PqWRvz</a> by Jason Brown (<a href='http://codepen.io/browniefed'>@browniefed</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

