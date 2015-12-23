---
layout: post
title: "React Native - How to bridge a Swift View"
date: 2015-11-28 11:09
comments: true
categories: react-native gradient swift bridge swift-view-component
facebook:
    image: http://i.imgur.com/TsOMRom.png
twitter_card:
    creator: browniefed
    type: summary_large_image
    image: http://i.imgur.com/TsOMRom.png
---

So in my previous article we bridged an Objective-C View component, I learned a lot, and also learned that I did a few unnecessary things. No worry though, we'll learn from that and make improvements in our Swift bridging.

First off want to give a thanks to [Tony Xiao](https://twitter.com/tony_xiao). He's been a great help in sharing code and giving explanations to a Swift and native development novice like myself. Also [Asa Miller](https://twitter.com/realasa) for writing some of the original code and always collaborating on whatever we're both working on.

## What

{% img http://i.imgur.com/TsOMRom.png Gradients, are those still a thing? %}

We're going to bridge [https://github.com/soffes/GradientView](https://github.com/soffes/GradientView). Looks pretty nifty. Now we could also just use ReactART for this but we won't.
<!-- more -->

## Why Swift?

It looks more like JavaScript, it's nicer to read (aka less brackets), and seemingly easier to write? I don't know, they say it's the next big thing.

## Why Not Swift?

Well it can't be turned into a static library... wut? It seemingly can't be converted into one of those nice `.xcodeproj` files with a `.a` that we can easily link. However it can be converted into an `.xcodeproj` with a `.framework` we can link, not sure what the difference is but whatever. We'll still go about bridging. We could have also used Pods here, but once again the point of this post is about bridging and not Pods. I'll focus on those later.

## Create a Project	

I won't walk us through bundling up anything into a library, I'll just walk you through developing and integrating a Swift library.

So go ahead and fire off a `react-native init GradientTest` to create an empty project.

Once that is done open up the project `ios/GradientTest.xcodeproj` in XCode.


## Add GradientView

Now lets pull in the `GradientView` library I had mentioned up above. I pulled it down and just put it in a `GradientView` folder in the `ios` directory.

Directory structure looks something like

{% img http://i.imgur.com/wzIGDYk.png Aint nothin but a directory structure img %}

We'll go through the same general process of linking the library as always.

Right click, select `Add Files to GradientTest` and find the `GradientView.xcodeproj` that we just pulled down.

When we go to link the library in `Build Phases` like normal it will actually be `GradientView.framework` that we are going to link. Should look something like so.

{% img http://i.imgur.com/6EIs7Vu.png It is all magic to me %}

We'll still get some issues so lets go add `$(SRCROOT)/GradientView` recursive to the `Framework Search Paths`
Should look something like

{% img http://i.imgur.com/bWs3LuJ.png I wish XCode was more magical %}

## Create our Manager and Bridging Header

We're going to first start off by creating a new `.swift` file. Simply do that by right clicking on our `GradientTest` folder, select `New file` and click on the swift selection and click create.

We'll call this file `RNGradientViewManager`. 

** CREATE THE BRIDGING HEADER**

{% img http://i.imgur.com/99mUsRS.png Bridge across troubled code %}


This is super important. Our briding header file will be named `GradientTest-Bridging-Header.h`. This allows us to bridge the React View Manager (Objective-C) code into the Swift world. 

Or as you can read in the file
```
//
//  Use this file to import your target's public headers that you would like to expose to Swift.
//

```

All we need to do is add our `RCTViewManager.h` import. So our `GradientTest-Bridging-Header.h` should just be this

```
//
//  Use this file to import your target's public headers that you would like to expose to Swift.
//

#import "RCTViewManager.h"
```


## Add The Manager Code

Okay now we need to add our Manager code, if you've read my previous tutorial before on bridging Objective-C, I've stated that the Manager is just a singleton View producer. Just because it is in swift is no different. 

`RNLinearGradientManager.swift`

```
import Foundation

@objc(RNLinearGradient)
class RNGradientViewManager : RCTViewManager {
  override func view() -> UIView! {
    return UIView(); // We'll change this later
  }
}
```

We create an override for the `view` function, this is what React will call to produce the new view. Thanks to our bridging header we get access to `RCTViewManager` and create a class called `RNGradientViewManager`.

The important thing to call out here is the `@objc(RNLinearGradientSwift)` line. This is an arbitrary name I made up, but what it says is to tell XCode and the compiler to expose the class `RNGradientViewManager` to the Objective-C world and call it `RNLinearGradientSwift`.


## The Objective-C React Native Part

Well at some point we have to dive into a little Objective-C. We'll create a new class and call it `RNLinearGradient`

{% img http://i.imgur.com/fcBMyIS.png How do you say cocoa %}

Then we'll subclass it off of `RCTView`

{% img http://i.imgur.com/57Vcf0j.png Generate code so I do not have to %}

That will create 2 files.

`RNLinearGradient.h`
`RNLinearGradient.m`

This is where we will tell React Native about what we need and what to call our stuff in the JavaScript world.

#### First our `RNLinearGradient.m` file.

```
#import "RNLinearGradient.h"
#import "RCTViewManager.h"

@interface RCT_EXTERN_MODULE(RNLinearGradientSwift, RCTViewManager)

RCT_EXPORT_VIEW_PROPERTY(locations, NSArray);
RCT_EXPORT_VIEW_PROPERTY(colors, NSArray);

@end
```
We import our `RNLinearGradient.h` file we will get ot in a second, as well as our `RCTViewManager.h`

See there is our Manager we created and called `RNLinearGradientSwift`.

We create a new interface and call `RCT_EXTERN_MODULE` which will tell the JavaScript world about our module called `RNLinearGradientSwift`.

We'll also export 2 view properties. These are specific to `GradientView`. Both will be `NSArray` and take an array of colors and locations to create the gradient.

#### Now our `RNLinearGradient.h` file.

```
#import "RCTView.h"

@interface RNLinearGradient : RCTView

@property (nonatomic, assign) NSArray *locations;
@property (nonatomic, assign) NSArray *colors;
@end

```
We import `RCTView` from React code, and then define our 2 properties.

Defining these properties here in this file tells React what properties to apply to the View that gets returned from our `view` function call in our `RNGradientViewManager`.


## Back to Swift 

Alright now we are back to swift. We'll go ahead and create a new swift file that will be the view that gets returned from the `view` function in our `RNGradientViewManager`.

Create a swift file called `RNGradientView`

```
import Foundation
import GradientView

@objc(RNGradientView)
class RNGradientView : GradientView {
  required init?(coder aDecoder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
  }
}
```

We'll import our `GradientView` library, and we'll create a class that inherits from it. Previously in my Objective-C tutorial we implemented the `addSubviews` call however here we'll implement things a bit different and have the view that gets returned the actual `GradientView`.

The whole `required init?(coder aDecoder: NSCoder)` part was autogenerated by XCode, it's all magic to me. 


Now lets add an init for the frame. The frame is the `CGRect` that we get given. It tells us our origin `x,y` and also the size `width,height` of our view.

```
@objc(RNGradientView)
class RNGradientView : GradientView {
  
  override init(frame: CGRect) {
    super.init(frame: frame);
    self.frame = frame;
  }
  
  required init?(coder aDecoder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
  }
}
```

We call `super.init` with our frame, and then also assign `self.frame = frame`. Remember because we inherited from `GradientView` the `self` basically refers to `GradientView` so when we want to manipulate the `GradientView` we simply manipulate our `self`.

Lets add some setters now. We defined 2 different properties up above `locations` and `colors`. We now need to create our setters so we can receive the values and set them.


```
@objc(RNGradientView)
class RNGradientView : GradientView {
  
  override init(frame: CGRect) {
    super.init(frame: frame);
    self.frame = frame;
  }
  
  required init?(coder aDecoder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
  }
  
  func setLocations(locations: NSArray) {
    self.locations = locations.map({ return $0 as! CGFloat});
  }
  
  func setColors(colors: NSArray) {
    self.colors = colors.map({return RCTConvert.UIColor($0)})
  }
}
```

Our `locations` is an `NSArray` of various values, they are sent over from the JavaScript world like so.

`<LinearGradient locations={[0.2, 1]} />`, however they will be a mix of `doubles` and `integers` but we want everything as a `CGFloat` which is what `GradientView` is expecting. 

From a JavaScript world you may be used to functional programming, we `map` over the array which will return a new array. Then we cast whatever values `double` or `integer`, etc as `CGFloat` and set that on `self.locations`.

Then for `colors` we'll do something similar. However in our JavaScript world we use the handy `processColors` provided to use by React. Which will process arbitrary `hex`, `rgb`, color names, and others into a value that can easily be passed over the bridge.

The JavaScript code could look something like `<LinearGradient colors={['red', 'rgb(255,100,50)']} />`.

We take advantage of the `RCTConvert` from React which gives us a bunch of handy ways to convert arbitrary values into other values. In our case `GradientView` expects `UIColor`s in the shape of an array. So we can once again `map` over our array of values and pass them into `RCTConvert.UIColor`.

## Fix Our Manager

Remember we just left our Manager to return a boring ole `UIView`, lets go fix it up to return our `RNGradientView`. 

```
import Foundation

@objc(RNLinearGradientSwift)
class RNGradientViewManager : RCTViewManager {
  override func view() -> UIView! {
    return RNGradientView();
  }
}
```
That sure was hard :)

## JavaScript code

We will need to write some JavaScript code but it should be pretty harmless.

```
import React, { requireNativeComponent, processColor } from 'react-native';
let RNLinearGradient = requireNativeComponent('RNLinearGradientSwift', LinearGradient);

class LinearGradient extends React.Component {
  render() {
    let { colors, ...otherProps } = this.props;
    return <RNLinearGradient {...otherProps} colors={processColor(colors)} />;
  }
}

LinearGradient.propTypes = {
  colors: React.PropTypes.array.isRequired,
  locations: React.PropTypes.array,
}

export default LinearGradient;
```

We bring in `React`, and our `requireNativeComponent` as well as `processColor` from React Native.

We create our `RNLinearGradient` component that we render, and as you can see once again there is our `RNLinearGradientSwift` that was externalized in our `RNGradientViewManager` as `RNLinearGradientSwift`.

We specify our `propTypes` as arrays. 

Then we can pull of `colors` from `this.props`, and we call `processColor` with the array which will automatically map over and return a new array of all processed colors that will work nicely with the `RCTConvert.UIColor` call.


## HOW DO I USE THIS THING?

```
        <LinearGradient 
          style={styles.gradient} 
          locations={[0, 1.0]} 
          colors={['#5ED2A0', '#339CB1']}
        />
```
To use we simply define some values. `locations` at `[0,1]` will specify the first color at 0 position and the second color at the end which is 1.
This can take more than just 2 colors and locations, it will take any number of colors and locations.

```
        <LinearGradient 
          style={styles.gradient} 
          locations={[0, .5, 1.0]} 
          colors={['#5ED2A0', 'red', '#339CB1']}
        />
```

Our gradient style is just postioned absolutely so it is basically covering the background as a gradient.

```
  gradient: {
    position: 'absolute',
    top: 0,
    left: 0,
    bottom: 0,
    right: 0,
  }
```


## DONE

Awesome, we're done! You can check the full code up here [https://github.com/asamiller/react-native-gradient](https://github.com/asamiller/react-native-gradient). Yes it's not on my github repo, it's on a friends repo which we collaborated on together. Deal with it.

{% img http://i.imgur.com/TsOMRom.png Gradients are neatoooo %}


