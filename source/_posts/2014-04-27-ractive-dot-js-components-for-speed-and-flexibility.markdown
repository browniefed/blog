---
layout: post
title: "Ractive.JS Components for Speed and Flexibility"
date: 2014-04-27 08:35
comments: true
categories: Ractive Ractive.js Components JavaScript
---


Components in Ractive are crucial if you want to build a flexible application. Hopefully this changes in the future with partials living on the data object and various init options accepting functions rather than static objects. The main purpose of Components is to have reusable template pieces that you can drop in and work the same all over your application. Some examples might be a grid component that accepts rows and columns as it's data, maybe it's as simple as a checkbox that has some styling a particular way. Components will help you build consistency through out your application however you should know when to use a partial and when to use a component.

If you're coming from the Angular world components are much like directives. However since there is no controller concept, or model concept in Ractive there are less headaches in getting components to work with scope and all that jazz.

Enough talk, lets jump in


### Example
{% raw %}

```js

var Checkbox = Ractive.extend({
	isolated: true,
	template: '<label><input type="checkbox" checked="{{checked}}"> {{label}}</label>',
	data: {
		checked: false
	}

});

```
{% endraw %}

<!-- more -->

To use it
{% raw %}

```js

var ractive = new Ractive({
	el: 'body',
	template: '<Checkbox checkbox="{{active}}" label="{{title}}" />',
	components: {
		Checkbox : Checkbox
	},
	data: {
		title: 'This is a title',
		active: true
	}
})
```
{% endraw %}


Here is the checkbox example I was talking about. Components are just another instiation of Ractive, using Ractive.extend says "use this stuff as the default". There are a few things to point on. 

On the component we have set `isolated` to `true`. This means that the template in the component does not have access to the parent data. That just means we couldn't do this.
{% raw %}

```js
template: '<label><input type="checkbox" checked="{{active}}"> {{title}}</label>'
```
{% endraw %}

In some cases you don't want your components to be isolated but for them to be modular and reusable it is good to make them isolated. Isolated is false by default.

Isolating your components means you are in control, you can name your data anything. As you see the parent Ractive has `active` and `title` but we still reference them as `checkbox` and `label` inside the component. Ractive will wire up the keypaths for you and bind everything. So when a user clicks on the checkbox `checked` will update to `false` or `true` and on the parent Ractive `active` will update to `true` or `false`, depending on if the checkbox is checked or unchecked.
Further more if you update the title, it'll propagate down to the component.

```js
ractive.set('title', 'This is a new title');
```

You are two-way data binding on DOM elements to the component as well as to the parent object. This is extremely powerful especially if you're coming from the jQuery world. There is no more finding the DOM element you want, determining if it is checked, finding the parent wrapping label, updating the text of the label. 


This is a very basic example. A more complex example would be the Grid component.

`grid-template`
{% raw %}


```html

<table>
	<tr>
		{{#columns}}
			<td>{{.label}}</td>
		{{/columns}}
	</tr>
	{{#rows:rowIndex}}
		<tr>
			{{#columns:columnIndex}}
				<td>
					{{rows[rowIndex][.field]}}
				</td>
			{{/columns}}
		</tr>
	{{/rows}}

</table>
```
{% endraw %}


```js

var Grid = Ractive.extend({
	isolated: true,
	template: '#grid-template',
	data: {}
});

```
{% raw %}

```js

var ractive = new Ractive({
	el: 'body',
	template: '<Grid rows="{{users}}" columns="{{cols}}" />',
	data: {
		cols: [
			{field: 'username', label: 'Username'},
			{field: 'name', label: 'Full Name'},
			{field: 'email', label: 'Email'}
		],
		users: [
			{username: 'admin', name: 'Admin', email: 'admin@example.com'},
			{username: 'tg', name: 'Thadius Gorge', email: 'tg@example.com'},
			{username: 'f999', name: 'Frank', email: 'frank@example.com'}

		]
	},
	components: {Grid: Grid}
})
```
{% endraw %}


Here is the live example


<p data-height="268" data-theme-id="0" data-slug-hash="dvqIl" data-default-tab="result" class='codepen'>See the Pen <a href='http://codepen.io/browniefed/pen/dvqIl/'>dvqIl</a> by browniefed (<a href='http://codepen.io/browniefed'>@browniefed</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//codepen.io/assets/embed/ei.js"></script>


All it takes is some slightly organized data and you'd never have to construct a data table again. More advance component topics are coming in the future this was merely an introduction.