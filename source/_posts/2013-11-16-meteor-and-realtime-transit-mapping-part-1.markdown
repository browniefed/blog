---
layout: post
title: "Meteor and Realtime Transit Mapping - Part 1"
date: 2013-11-16 09:00
comments: true
categories: meteor meteor.js trimet maps relatime transit websockets
---

#Overview
The point of this series is to help people build out realtime transit maps for other cities.
This will focus on Portland, OR and their transit system TriMet. They give us access to a wealth of information but all we really need is route numbers that make sense and latitude/longitude to make a map. I have built one for Kansas City, MO but they have their data hidden. Don't be afraid to dig around on your own transit website to find hidden APIs.

Finally we chose MeteorJS because the whole system is built on realtime applications. Code can be run on both the client/server which is AWESOME! It pretends to be lo-latency, meaning it'll save the data in client side minimongo databases and not wait for the server to return "yeah we saved". It also somehow magically invalidates the data correctly. It's just a framework that works and I can't wait for the 1.0 release.

As always all code is open sourced. Check out the final project here
https://github.com/browniefed/livemet

#Setup

##APIs

Go to http://developer.trimet.org/registration/ and get an API key.
The documentation we care about is here http://developer.trimet.org/ws_docs/

## Meteor

Currently on Mac/Linux are officially supported and Windows has an unofficial installer. But you can't use Meteorite on Windows which you want meteorite!
To install simple go here and follow the instrutions(2 steps!) http://docs.meteor.com/
Step 2 being creating a meteor project

```
meteor create realtimetransit
cd realtimetransit
```
Now that you have a meteor app created, change directory into so when you execute commands it actually does stuff to your meteor project.

##Meteorite
Install meteorite. I'm assuming you have node/npm installed. If not go install it so you can actually use modern tooling!
Simply run (you may not need the sudo -H)

```
sudo -H npm install -g meteorite
```
For more info and all meteor packages visit https://atmosphere.meteor.com/wtf/app

#Step 1 - Create some Boilerplate

Meteor has a specific folder structure that it uses to load the server/client/public data.
Delete the stuff they auto-generated for you, we do not care about it.
Then go ahead and create this folder structure in your meteor app.

```
client
-css
-helpers
-js
-sass(I setup a compass project but you do not have to)
-views
-main.html
-main.js
collections
server
```
Read more about this file structure here http://docs.meteor.com/#structuringyourapp

#Step 2 - Get some templates created

If you have meteorite installed add the leaflet package. We are going to use leafletjs + Open Stree Maps because they're both AWESOME.
You could also create your own map tiles using TileMill, or use MapBox for hosted tiles but that is a topic for another time.

Run

```
mrt install leaflet
```

Meteor is special in that it takes ALL html files in specified directories, bundles them up and turns them into Handlebar templates.
`main.html` is going to be our base template, this will load another template called body, which will load other templates.
This is what we call structure.

```html
{% raw %}
<head>
	<title>PDXLiveBus - Trimet Realtime</title>
	<meta name="viewport" content="width=device-width,initial-scale=1">
</head>
<body>
	{{> body}}
</body>
{% endraw %}
```

Now lets create the body template in the `client/views` foler
Each template is to be wrapped in a template tag and given a name so that meteor can create a reference to it.

```html
{% raw %}
<template name="body">
{{> map}}
</template>
{% endraw %}
```
Hey look, now it wants a map template, lets create one.

```html
{% raw %}
<template name="map">
	{{# constant}}
	<div class="map_container" id="map_container">
	</div>
	{{/constant}}
</template>
{% endraw %}
```
The `{% raw %} {{#constant}}{% endraw %}` tag tells meteor that it should ignore it's auto updating features when data changes and leave this template portion alone.
If we didn't then our map would be constantly be refreshing if we happened to have a parent template with data that changed. We do not in this case but it's best to add it.
There are improvements being made to meteor so that you won't have to do this in the future.

#Step 3 - Get a map rendering

Now we're going to get into some meteor javascript. 

```javascript
Template.map.rendered = function() {}
```
The Template is a global object where all templates get loaded. The map is the name of our map template.
Because Meteor renders a series of templates we are unsure when the DOM is going to be ready.
They give us the rendered function which means the DOM for this template is now ready to be worked with.
What that means for us is that we now have a container to add a map to.


Lets add a basic map to it with Open Street Map tiles

``` javascript
Template.map.rendered = function() {

		var w = window.innerWidth;
		var h = window.innerHeight;
		$('#map_container').css({width: w+'px', height: h+'px'});
	   	var map = L.map('map_container', {maxZoom: 19, zoom: 13, zoomControl: false, center: ['45.525292','-122.668197']});
		map.attributionControl.setPrefix('');
		L.tileLayer('http://{s}.tile.osm.org/{z}/{x}/{y}.png').addTo(map);
		L.Icon.Default.imagePath = 'packages/leaflet/images';
}
```

Because we are unsure of how big the the users window is going to be (destkop/tablet/phone/etc) we will just set the map container to that width.
Ours just happens to be the full page.

Because the way meteor works L was defined globally on the client side when we included that package with meteorite.
So we'll create a map by passing in our `map_container` ID that we know is ready to use, and center it in Portland. 
The other parameters are fairly self explanatory and well documented on Leaflets website.

We remove the attribution from the map but we'll add it back in elsewhere. This just removes the tribute to Leaflet for it's awesome library.

Finally we set a tilelayer. This is a standard tile map link.

```
{s} = Subdomains to get tiles from(this parameter is just to get around browser limitations on simultaneous HTTP connections to a single host)
{z} = Zoom level
{x} = X Coordinate
{y} = Y Coordinate
```

A deeper exploration into how tile servers work can be found here http://wiki.openstreetmap.org/wiki/Slippy_map_tilenames#Tile_servers

Now we can run meteor and all should be well.
You should have a map that completely fills your window.

Part 2 getting Trimet data.
Part 3 adding markers and animating their movements