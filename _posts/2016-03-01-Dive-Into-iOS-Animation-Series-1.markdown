---
layout: post
title: "Dive Into iOS Animation 1"
date: 2016-03-01 00:39:49 +1000
comments: true
header-img: "img/post-bg-12.jpg"
categories: [ios, animation]
reading_time: "15 mins"
---

I've been fiddling with Core Animation for a while, it's time for me to write down those gems I've discovered during the journey.
So let's get started with our first post in animation series, what will happen when you call `UIView.animateWithDuration` ?

<!--more-->

In order for us to understand what's going on about UIView helper `animateWithDuration`, we need to setup a tiny demo.
Here we have a rectangle starting at top left corner of the screen. Then we animate rectangle to move towards right bottom a little bit, meanwhile, the background color of its superview gets changed from green to gray.

![demo](/img/blog/iOS/core_animation/demo_1.gif)

{% gist 5dbeb12d8fdb06e8207bea5247637a0f core_animation_demo_1.swift %}

If we put breakpoint from line 4 to line 10, it is quite clear for us that the order of code execution is as follows:

1. line 4: superview's background color sets to green
2. line 6: we are about to call `animateWithDuration`
3. line 9: we will first jump to ending brace due to `animateWithDuration` takes closure as parameter
4. line 7: now we are inside the closure, it seems that we are setting background color and center directly

## Question 1

Question is, is that what actually happening ?

Because clearly when we see the demo, it looks that the animation takes some time. It doesn't take place instantly, the color doesn't immediately change to gray, so with the position of the rectangle.

The answer is when you set the background color or the center or any property that's animatable, yes, they do immediately change on the view.

We can verify that by printing out the background color before and after the change.

![demo](/img/blog/iOS/core_animation/core_animation_demo_1.png)

## Question 2

Here we get another question: how is that possible that view is still animating given the property get instantly changed here ?

The answer to this question actually lies in the core animation architecture itself.

`animateWithDuration` is actually a wrapper around core animation. So when you set the background color or any animatable property, you are actually setting these values, no doubt about that. However, you are setting these values in something called core animation's *layer tree*, or sometimes called *model tree*.

![core animation architecture](/img/blog/iOS/core_animation/core_animation_architecture.png)

When you set or get any animatable property, you are actually setting or getting these values from *model tree*. And when you start animation, the *model tree* get immediately copied and swapped with *presentation tree*. Then this *presentation tree* contains current animating values.

That's why when you query the background color it seems like ending value, only *presentation tree* contains current value on the screen.

## Take Home
- UIView's animation helper is a thin wrapper for core animation API
- *Model tree* is used for setting and getting the animatable properties
- *Presentation tree* contains animatable values that is actually rendering on the screen
- Knowing the underlying architecture opens the door to explore the advanced core animation APIs :)

## Reference
- [Core Animation Programming Guide](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/CoreAnimationBasics/CoreAnimationBasics.html#//apple_ref/doc/uid/TP40004514-CH2-SW3)
- [WWDC2014 Building Interruptible and Responsive Interactions](https://developer.apple.com/videos/play/wwdc2014/236/)
