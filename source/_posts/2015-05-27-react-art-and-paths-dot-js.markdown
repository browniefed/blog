---
layout: post
title: "React Art and Paths.js"
date: 2015-05-27 19:32
comments: true
categories: react react-art paths.js svg canvas graphing
---

### Intro

[Paths.js](https://github.com/andreaferretti/paths-js) is a cool library. It comes with 3 levels of generating paths. 

* Low level which helps you generate paths/lines.
* Mid level which generates paths for shapes
* High level which takes a set of data and generates graphs

All of these are great when working with `react-art` because it's just returning data. I'll say it once, and a million more but libraries that just generate data make it easy to traverse and render with `react-art`.

<!-- more -->


### Paths

#### UPDATE: 

Sebastian Markbage (the creator of ReactART and ART), informed me that `ReactART` itself has a `Path` implementation. I realized this but didn't think much of it.
However per the discussion here [https://discuss.reactjs.org/t/react-art-with-paths-js/492](https://discuss.reactjs.org/t/react-art-with-paths-js/492) the `ART` path is faster as it takes advantage of the current `mode` to create the most efficient path instead of taking a string and converting it back into native for instructions for canvas rendering.

The `ReactART.Path` has similar methods. Check out the discuss thread for a link to the implementation to find out the supported methods. I'm currently working on `ReactART` documentation so expect that soon.


Example:

```
var path = Path()
  .moveto(10, 20)
  .lineto(30, 50)
  .lineto(25, 28)
  .qcurveto(27, 30, 32, 27)
  .closepath();
```


This is can just be plugged right into `react-art` `Shape` element.

Like so

```
var React = require('react'),
	ReactArt = require('react-art'),
	Surface = ReactArt.Surface,
	Shape = ReactArt.Shape,
	Path = require('paths-js/path');

var Demo = React.createClass({
	getInitialState: function() {
		return {
			to: {
				x: 30,
				y: 50
			}
		}
	},
	getPath: function() {
		var path = Path()
					  .moveto(10, 20)
					  .lineto(this.state.to.x, this.state.to.y)
					  .lineto(25, 28)
					  .qcurveto(27, 30, 32, 27)
					  .closepath();

	 	return path.print();
	},
	startAnimating: function() {

		if (this.state.to.x === 100) {
			this.addToPosition = -1;
		} else if (this.state.to.x === 29) {
			this.addToPosition = 1;
		}

		this.state.to.x += this.addToPosition;
		this.state.to.y += this.addToPosition;

		this.setState(this.state);
	},
	componentDidMount: function() {
		this.addToPosition = 1;
		setInterval(this.startAnimating, 17)
	},
	render: function() {
		return (
			<div>
				<Surface
					width={500}
					height={500}
				>
					<Shape d={this.getPath()} stroke="#000" strokeWidth={1} />
				</Surface>
			</div>
		)
	}
});

module.exports = Demo;
```

<p data-height="624" data-theme-id="0" data-slug-hash="VLmOOE" data-default-tab="result" data-user="browniefed" class='codepen'>See the Pen <a href='http://codepen.io/browniefed/pen/VLmOOE/'>VLmOOE</a> by Jason Brown (<a href='http://codepen.io/browniefed'>@browniefed</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

### Simple Shapes

Now `react-art` already comes with a few different shapes but `paths.js` have a few built in as well, like `Rectangle` and `Bezier` curve.

```
var rectangle = Rectangle({
  top: 10,
  bottom: 3,
  left: -2,
  right: 5
});
```

And how that looks is very similar to the previous example


```
var React = require('react'),
	ReactArt = require('react-art'),
	Surface = ReactArt.Surface,
	Group = ReactArt.Group,
	Shape = ReactArt.Shape,
	Rectangle = require('paths-js/rectangle'),
	Bezier = require('paths-js/bezier');

var Demo = React.createClass({
	getPath: function() {
		var rectangle = Rectangle({
					  top: 10,
					  bottom: 3,
					  left: -2,
					  right: 5
					});

	 	return rectangle.path.print();
	},
	getBez: function() {
		var points = [[1, 50], [50, 100], [100, 3], [4, 0]];
		var curve = Bezier({
		  points: points,
		  tension: 0.2
		});

		return curve.path.print();
	},
	render: function() {
		return (
			<div>
				<Surface
					width={500}
					height={500}
				>
					<Group x={100} y={100}>
						<Shape d={this.getPath()} stroke="#000" strokeWidth={1} />
					</Group>
					<Group x={200} y={200}>
						<Shape d={this.getBez()} stroke="#000" strokeWidth={1} />
					</Group>
				</Surface>
			</div>
		)
	}
});

module.exports = Demo;
```

<p data-height="624" data-theme-id="0" data-slug-hash="xGRNvW" data-default-tab="result" data-user="browniefed" class='codepen'>See the Pen <a href='http://codepen.io/browniefed/pen/xGRNvW/'>xGRNvW</a> by Jason Brown (<a href='http://codepen.io/browniefed'>@browniefed</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>


### Graphs


```
var pie = Pie({
  data: [
    { name: 'Italy', population: 59859996 },
    { name: 'Mexico', population: 118395054 },
    { name: 'France', population: 65806000 },
    { name: 'Argentina', population: 40117096 },
    { name: 'Japan', population: 127290000 }
  ],
  accessor: function(x) { return x.population; },
  compute: {
    color: function(i) { return somePalette[i]; }
  },
  center: [20, 15],
  r: 30,
  R: 50
});
```

Some code of it in action

```
var React = require('react'),
	ReactArt = require('react-art'),
	Surface = ReactArt.Surface,
	Group = ReactArt.Group,
	Shape = ReactArt.Shape,
	Pie = require('paths-js/Pie');


var pie = Pie({
  data: [
    { name: 'Italy', population: 59859996 },
    { name: 'Mexico', population: 118395054 },
    { name: 'France', population: 65806000 },
    { name: 'Argentina', population: 40117096 },
    { name: 'Japan', population: 127290000 }
  ],
  accessor: function(x) { return x.population; },
  compute: {
    color: function(i) { return '#000'; }
  },
  center: [20, 15],
  r: 30,
  R: 50
});

var Demo = React.createClass({

	getPie: function() {
		return pie.curves.map(function(shape) {
			return (
				<Group>
					<Text fill="#A6BD8A" font='bold 12px "Arial"' x={shape.sector.centroid[0] - 12} y={shape.sector.centroid[1]}>{shape.item.name}</Text>
					<Shape d={shape.sector.path.print()} stroke={shape.color} strokeWidth={1} />
				</Group>
			)
		})
	},
	render: function() {
		return (
			<div>
				<Surface
					width={500}
					height={500}
				>
					<Group x={50} y={45}>
						{this.getPie()}
					</Group>
				</Surface>
			</div>
		)
	}
});


module.exports = Demo;
```

<p data-height="624" data-theme-id="0" data-slug-hash="waoLvB" data-default-tab="result" data-user="browniefed" class='codepen'>See the Pen <a href='http://codepen.io/browniefed/pen/waoLvB/'>waoLvB</a> by Jason Brown (<a href='http://codepen.io/browniefed'>@browniefed</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

### Conclusion

These examples may look boring but they just show off a bit of the control you can have with `react-art` and a simple path generator.
Not only that but because we aren't depending on the DOM in any case these examples should also work on `react-native`.
Combined with some tweening you could make some very effective graphs that animate. That is a topic for another time.