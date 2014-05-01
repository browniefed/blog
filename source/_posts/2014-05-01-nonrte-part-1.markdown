---
layout: post
title: "NonRTE - Part 1"
date: 2014-05-01 06:26
comments: true
categories: javascript nonrte contenteditable
---

Once again stealing from antimatter I think writing about what you're currently building or have built is a good way to gain some clarity and also provide some value in explaining the topic. I'll be talking about NonRTE a non-contenteditable rich text editor for developers. One similar and the only one I know of is google docs. There are others that aren't rich text editors and are IDEs like codemirror and a few others that do not use contenteditbale. I personally don't know of any RTEs that are open source that developers can use and build with but I could be wrong so do ping me if that isn't the case.

Something that has always boggled my mind was how heavily people have relied on contenteditable. Managing the different ways that contenteditbale is implemented on every browser and versions of those browsers is a nightmare. There is a portion of code in most RTE libraries that change how everything works just for one particular browser. This is insane to me due to the world we live in now with NPM, github, and the open source world at our finger tips. We have amazing templating libraries, HTML parsers, two-way data bindings, word to HTML libraries, excel expression parsers and executors, and everything compiles to JavaScript despite this insane amount of open source code we still use contenteditable.

These aren't all the components that necessarily build an RTE you have to think about the cursor and positioning it correctly, selection and how to implement that, styling the complexities of every possible selection, resizing the container and line widths no longer being the same (this is the toughest one I think), copy and pasting, selection then clicking and dragging, and many many more components. These all sound hard but once again, if you look around you'll find varying levels of complete, incomplete, good, bad, popular, and unpopular repos that accomplish much of this. It sounds like I'm saying rip them off, I'm definitely not condoning that so be sure and give credit where credit is due.

I'm sure others have embarked on this particular quest only to fail or give up but I'll do my best not to. I see a lot of potential in this idea, many of the concepts are also very similar to Ractive. In fact I could implement this even faster in Ractive but have decided to embark on this from scratch just to learn things. It'll be a fun ride. 

At this point I have very basic typing working, the cursor is working and positioned (mostly) correctly, clicking somewhere will move the cursor, arrow keys will move from line to line and character to character correctly and enter to create new lines and split content is also working. Overall it's a little bit of work but nothing quite substantial yet. The repo lives here [https://github.com/browniefed/Nonrte](https://github.com/browniefed/Nonrte) so feel free to contribute. I haven't hit many challenges yet but I will and when I do I'll be sure and document them and document the solutions.