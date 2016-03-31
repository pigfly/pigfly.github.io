---
layout: post
title: "Dive Into iOS Animation 3"
date: 2016-03-30 21:39:49 +1000
comments: true
header-img: "img/post-bg-14.jpg"
categories: [ios, animation]
reading_time: "30 mins"
---

Remember when we want to set the rectangle back in the [previous post](http://pigfly.github.io/ios/animation/2016/03/26/Dive-Into-iOS-Animation-Series-2/) ? We fail to do that because we didn't create animation transaction for that position change.  In this post, we will explore three different ways of how Cocoa manages this.

<!--more-->

Before we investigate any further, let's take a look at this demo first.

![demo](/img/blog/iOS/core_animation/core_animation_demo_3.gif)

We want to allow user to cancel animation even before it finishes. That is being responsive, to build a good user experience. Here we get three different situations.

1. There is a big jump for red rectangle to complete these two animations
2. The green one is slightly better. However, it actually feels like running into the wall, lost the original velocity
3. The blue one is the best behavior, it has smooth animation from the beginning to the end

Actually, these three animations represent three different ways of handling animations in iOS. Let me walk you through using the example below.

![demo](/img/blog/iOS/core_animation/demo_3_2.gif)

Here we want the circle to animate from left to right, then in the middle of the animation, we add another animation to bring it back to origin.

## First

In order for us to understand what is going on behind the scene, we need to know a little detail about how Cocoa manages animations in general.

Every time you set animatable properties you are actually creating a `CAAnimation` object, it is operating on the `CALayer` level. However you don't need to deal with all these details since UIKit takes care for you. So if you want to animate the center, say, `demoView.center = CGPointMake(500, 0);`, UIKit will create a `CAAnimation` like this:

| CAAnimation |        |
|-------------|--------|
| fromValue   | (0, 0) |
| duration    | 1.0    |
| beginTime   | 1000.1 |

1. The fromValue is copied from current model value, since at 1000.0, the model value is (0, 0)
2. The duration indicates how long this animation gonna take

If we read this piece of information in natural language, it will be "this is the animation gonna animate from position (0, 0) to the current model value over 1 second staring from 1000.1". In our case, the next model value will be 500. If you are confused about the model value or presentation value here, check out my very [first post](http://pigfly.github.io/ios/animation/2016/03/01/Dive-Into-iOS-Animation-Series-1/) for details.

Armed with this `CAAnimation` object, we can now observe the changes both in model layer and presentation layer.

| Time               | 1000.0 | 1000.1   | 1000.2   | 1000.3   | 1000.4   | 1000.5   |
|--------------------|--------|----------|----------|----------|----------|----------|
| Model value        | (0, 0) | (500, 0) | (500, 0) | (500, 0) | (500, 0) | start    |
| Animation value    | n/a    | (0, 0)   | (50, 0)  | (100, 0) | (150, 0) | reverse  |
| Presentation value | (0, 0) | (0, 0)   | (50, 0)  | (100, 0) | (150, 0) | animation|

1. The `CAAnimation` object remains the same until we are about to start reverse animation at 1000.5
2. The model value reflects the animation destination value or `toValue` for `CAAnimation`
3. The presentation values reflects the actual rendering value on the screen

Every thing seems fine until this point. However as we set the center `demoView.center = CGPointMake(0, 0)` back to origin at 1000.5, things become more interesting.

Remember the very first `CAAnimation` object we created ? It get destroyed with the replacement of new `CAAnimation` object. Why ? Because we are setting the animatable property again. Let's repeat one more time, setting animatable property equals to create a new `CAAnimation` object.

| CAAnimation |          |
|-------------|----------|
| fromValue   | (500, 0) |
| duration    | 1.0      |
| beginTime   | 1000.5   |

Notice how this sudden change affects the values in the presentation layer:

| Time               | 1000.0 | 1000.1   | 1000.2   | 1000.3   | 1000.4   | 1000.5   | 1000.6   | 1000.7   |
|--------------------|--------|----------|----------|----------|----------|----------|----------|----------|
| Model value        | (0, 0) | (500, 0) | (500, 0) | (500, 0) | (500, 0) | (0, 0)   | (0, 0)   | (0, 0)   |
| Animation value    | n/a    | (0, 0)   | (50, 0)  | (100, 0) | (150, 0) | (500, 0) | (450, 0) | (400, 0) |
| Presentation value | (0, 0) | (0, 0)   | (50, 0)  | (100, 0) | (150, 0) | (500, 0) | (450, 0) | (400, 0) |

So starting from 1000.4, we will animate from 500 back to 0. However, the value for the presentation layer jumps from 150 to 500 within 0.1 second. That's why we see the sudden leap for the first ball !

## Second

The `BeginFromCurrentState` is slightly different with the previous one. Instead of setting the `fromValue` for the new `CAAnimation` object to the current model value, we set it to be the current presentation value.

| CAAnimation |          |
|-------------|----------|
| fromValue   | (150, 0) |
| duration    | 1.0      |
| beginTime   | 1000.5   |

So there is no sudden value change at presentation layer any more. That means you won't experience any jump for animation !

| Time               | 1000.0 | 1000.1   | 1000.2   | 1000.3   | 1000.4   | 1000.5   |
|--------------------|--------|----------|----------|----------|----------|----------|
| Model value        | (0, 0) | (500, 0) | (500, 0) | (500, 0) | (500, 0) | (0, 0)   |
| Animation value    | n/a    | (0, 0)   | (50, 0)  | (100, 0) | (150, 0) | (150, 0) |
| Presentation value | (0, 0) | (0, 0)   | (50, 0)  | (100, 0) | (150, 0) | (150, 0) |

However, now we get another problem. It is the root of causing hitting the wall and losing velocity bug for the second circle.

![demo](/img/blog/iOS/core_animation/begin_from_current_state.png)

This is a snapshot for our animating circle. Notice the reverse animation, we are animating the smaller distance over the same amount of time.
That's gonna create problem for us since these two circle's velocity is not the same !

## Third

We've already known that `beginFromCurrentState` solves the sudden position change in presentation layer. However, it introduces another problem of sudden velocity change. Is there a solution for solving both ? Yes, that is additive animation.

- In additive animation, `fromvalue` and `toValue` are interpreted relatively to the model value
- presentation value = model value + animation value
- we don't destroy `CAAnimation` every time we create a new one. Instead, we add new `CAAnimation` to the old one.

| CAAnimation1|               |    | CAAnimation2  |             |
|-------------|---------------|    |---------------|-------------|
| additive    | TRUE          |    | additive      |  TRUE       |
| fromValue   | (-500, 0)     |    | fromValue     |  (500, 0)   |
| toValue     | (0, 0)        |    | toValue       |  (0, 0)     |
| duration    | 1.0           |    | duration      |   0.4       |
| beginTime   | 1000.1        |    | beginTime     |   1000.5    |

And we add those values in each layer as well:

| Time               | 1000.0 | 1000.1    | 1000.2    | 1000.3    | 1000.4    | 1000.5    |
|--------------------|--------|-----------|-----------|-----------|-----------|-----------|
| Model value        | (0, 0) | (500, 0)  | (500, 0)  | (500, 0)  | (500, 0)  | (0, 0)    |
| Animation 1        | n/a    | (-500, 0) | (-450, 0) | (-400, 0) | (-350, 0) | (-300, 0) |
| Animation 2        | n/a    | n/a       | n/a       | n/a       | n/a       | (500, 0)  |
| Presentation value | (0, 0) | (0, 0)    | (50, 0)   | (100, 0)  | (150, 0)  | (200, 0)  |

So right now the presentation value = model value + animation 1 value + animation 2 value. And you can see here the presentation value get changed smoothly without losing the original velocity.

The whole idea behind the additive animation is to make relative animation, that is, becoming path independent.

## Take Home
- Whenever you set animatable property, you create a `CAAnimation` object
- Old `CAAnimation` will be destroyed with the replacement of new one unless you are using additive animation
- Additive animation is the default behavior for iOS 8+
- Not all properties support additive animation, if not, use `beginFromCurrentState`
- To build better user experience, you need to avoid sudden changes of velocity and position in your custom animation

## Reference
- [WWDC2014 Building Interruptible and Responsive Interactions](https://developer.apple.com/videos/play/wwdc2014/236/)
- [Core Animation Programming Guide](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/CoreAnimationBasics/CoreAnimationBasics.html#//apple_ref/doc/uid/TP40004514-CH2-SW3)
- [CALayer Reference](https://developer.apple.com/library/mac/documentation/GraphicsImaging/Reference/CALayer_class/)
