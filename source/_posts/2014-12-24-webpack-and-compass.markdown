---
layout: post
title: "Webpack and Compass"
date: 2014-12-24 10:46
comments: true
categories: webpack compass sass scss
---

Webpack is the greatest and latest in build systems. Every time I encounter an issue or wish Webpack did something a little googling solves the problem.

One problem that I couldn't google and figure out was getting compass to work with `sass-loader`. Not just making it work but still allowing regular compass to compile on the command line.

Thankfully there is a mostly simple solution. The only thing (to my knowledge) that this won't work with is sprite generation. Sounds like we need another loader.

Lets get to it.

This is all assuming you have webpack installed with a basic config. 
If that isn't the case we'll need `webpack`, `raw-loader`, `style-loader`, and `sass-loader` npm installed.

Your webpack config may look something along the lines of 

```
        module: {
            loaders: [
                { 
                    test: /\.scss$/, 
                    loader: "style-loader!raw-loader!sass-loader"
                }
            ]
        }

```

This is where you're running into your issue where `node-sass` can't find the compass import.

Thanks to igosuki, compass mixins has been ported to just a bunch of sass-mixins outside of the ruby gem. [https://github.com/Igosuki/compass-mixins](https://github.com/Igosuki/compass-mixins)

So now do a 

```
npm install compass-mixins --save

```

Now we just need to modify our loader config.

```
        module: {
            loaders: [
                { 
                    test: /\.scss$/, 
                    loader: "style-loader!raw-loader!sass-loader?includePaths[]=" + path.resolve(__dirname, "./node_modules/compass-mixins/lib")
                }
            ]
        }

```

Now `node-sass` should be all setup to to look in our `compass-mixins/lib` to resolve compass mixins.
If you are running your build script from a different location you'll have to adjust the `path.resolve` to resolve to wherever your `node_modules` is at.

Alternatively you can place this outside of `node_modules` or add additional `include_paths` for wherever your SASS mixins live.

```
        module: {
            loaders: [
                { 
                    test: /\.scss$/, 
                    loader: "style-loader!raw-loader!sass-loader?includePaths[]=" + path.resolve(__dirname, "./node_modules/compass-mixins/lib") + "&includePaths[]=" + path.resolve(__dirname, "./mixins/app_mixins")
                }
            ]
        }

```
