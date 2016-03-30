---
layout: post
title: "Dive Into iOS Animation 2"
date: 2016-03-26 10:39:49 +1000
comments: true
header-img: "img/post-bg-13.jpg"
categories: [ios, animation]
reading_time: "10 mins"
---

In [previous post](http://pigfly.github.io/ios/animation/2016/03/01/Dive-Into-iOS-Animation-Series-1/) we've already known there are *model tree* and *presentation tree* that secretly working for core animation. In this post, we will take a look at two typical examples raising from these concepts.

<!--more-->

## First

The first example involves small changes to our previous code. After we animate rectangle to move towards right bottom, we want to move the rectangle back to the top left corner.

{% gist 3407e38bd36b9821e4d19a951150eebe core_animation_demo_2.swift %}

You may think that rectangle will get animated and moved to top left corner just as before. Since we are setting the center `self.demoView.center = CGPointMake(50, 50)` just like we did in line 9.

You wish...

![demo](/img/blog/iOS/core_animation/demo_2.gif)

What happened ? Why there is no animation for setting the rectangle back ?

Armed with the knowledge of core animation architecture, you should have a clue now. Yes, after the animation get completed, the *presentation tree* get released. Which, only leave the *model tree* in the memory. So what you are doing inside the completion closure is setting and getting properties in the *model tree*, has nothing to do with the animation on the screen. Remember remember, it is the *presentation tree* that is in charge of actual animation on screen.

## Second

Moving around rectangle seems boring, how about adding user interaction during the animation :)

So we change the UIView to UIButton, and hope we can click the button while the it's animating.

![demo](/img/blog/iOS/core_animation/demo_2_2.gif)

The circle indicates mouse clicking in the picture above. However, we don't have much luck since the button doesn't respond to click at all...

Is there something to do with the *model or presentation tree* again ? Bingo!

Technically speaking, as the view is animating, its `hitTest` is based on the *model layer* not the *presentation layer*. You can think of `hitTest` is the primitive function provided by `CALayer` which capturing user's clicking. What does that mean is when the user is trying to hit the button that is animating, they won't be able to tap the button unless they tap the location of the completed animation.

![demo](/img/blog/iOS/core_animation/demo_2_3.gif)

However, we can override the `hitTest` function to be based on the *presentation layer*. Then we can finally click the rectangle while it's animating.

{% gist cc9d7ca4c69aec870d1d0f3c8c40dd4d core_animation_demo_2_2.swift %}

## Take Home
- User interaction for animation is based on the *model layer*, and is disabled by default when animation get started.
- *Model tree* is used for setting and getting the animatable properties
- *Presentation tree* contains animatable values that is actually rendering on the screen
- Knowing the underlying architecture opens the door to explore the advanced core animation APIs :)

## Reference
- [Core Animation Programming Guide](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/CoreAnimationBasics/CoreAnimationBasics.html#//apple_ref/doc/uid/TP40004514-CH2-SW3)
- [CALayer Reference](https://developer.apple.com/library/mac/documentation/GraphicsImaging/Reference/CALayer_class/)
