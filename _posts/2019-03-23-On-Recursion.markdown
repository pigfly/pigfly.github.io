---
layout: post
title: "On Recursion"
date: 2019-03-23 11:02:30 +1000
comments: true
header-img: "img/post-bg-12.jpg"
categories: [software-engineering, recursion]
reading_time: "20 mins"
---

These series are the reflection and study when I was leading an engineering team for all the aspects of software development.
In this post, we will explore an important technique, *recursion* 🌿, which, it's a concept that many people scratch their
heads over.

<!--more-->

A recursive function is defined in terms of *base cases* and *recursive steps*:

1. In a base case, we compute the result immediately given the inputs to the function call
2. In a recursive step, we compute the result with the help of one or more recursive calls to this same function, 
   but with the inputs somehow reduced in size or complexity, closer to a base case.

Consider writing a function to compute factorial:

## The factorial

![demo](/img/se/factorial-def.png)

And we have the recursive implementation in Swift:

```swift
public static func factorial(n: Int) -> Int {
  if n == 0 {
    return 1
  } else {
    return n * factorial(n-1)
  }
}
```

For the above implementation, the recursive step is n > 0 , where we compute the result with the help of a recursive call to obtain (n-1)!

To visualize the execution of a recursive function, let's diagram🎨 the call stack of currently-executing functions as the computation proceeds.

Suppose we run the `factorial` function inside `main` thread as follows:

```swift
let x: Int = factorial(3)
```

<img src="/img/se/factorial-stacktrace.png" width="1200" height="600">

In the diagram, we can see how the stack grows:

1. `main` calls `factorial`
2. `factorial` then calls itself , until `factorial(0)` does not make a recursive call
3. the call stack unwinds, each call to factorial returning its answer to the caller, until `factorial(3)` returns to main

Here’s an [interactive visualization of factorial](http://www.pythontutor.com/visualize.html#mode=display). 
You can step through the computation to see the recursion in action. 

## TBD..
