---
layout: post
title: "On Recursion 1"
date: 2019-03-23 11:02:30 +1000
comments: true
header-img: "img/post-bg-12.jpg"
categories: [software-engineering, recursion]
reading_time: "15 mins"
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

<div class="overflow:scroll" style="margin-right: -25%;">
       <table class="table">
        <thead>
         <tr>
          <td>
           <p>
            starts in
            <br>
             <code>
              main
             </code>
            <br>
           </p>
          </td>
          <td>
           <p>
            calls
            <br>
             <code>
              factorial(3)
             </code>
            <br>
           </p>
          </td>
          <td>
           <p>
            calls
            <br>
             <code>
              factorial(2)
             </code>
            <br>
           </p>
          </td>
          <td>
           <p>
            calls
            <br>
             <code>
              factorial(1)
             </code>
            <br>
           </p>
          </td>
          <td>
           <p>
            calls
            <br>
             <code>
              factorial(0)
             </code>
            <br>
           </p>
          </td>
          <td>
           <p>
            returns to
            <br>
             <code>
              factorial(1)
             </code>
            <br>
           </p>
          </td>
          <td>
           <p>
            returns to
            <br>
             <code>
              factorial(2)
             </code>
            <br>
           </p>
          </td>
          <td>
           <p>
            returns to
            <br>
             <code>
              factorial(3)
             </code>
            <br>
           </p>
          </td>
          <td>
           <p>
            returns to
            <br>
             <code>
              main
             </code>
            <br>
           </p>
          </td>
         </tr>
        </thead>
        <tbody class="no-markdown">
         <tr>
          <td style="vertical-align: bottom;">
           <div class="panel panel-primary">
            <div class="panel-heading">
             main
            </div>
            <div class="panel-body">
            </div>
           </div>
          </td>
          <td style="vertical-align: bottom;">
           <div class="panel panel-primary">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 3
             <br>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             main
            </div>
            <div class="panel-body">
             x
            </div>
           </div>
          </td>
          <td style="vertical-align: bottom;">
           <div class="panel panel-primary">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 2
             <br>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 3
             <br>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             main
            </div>
            <div class="panel-body">
             x
            </div>
           </div>
          </td>
          <td style="vertical-align: bottom;">
           <div class="panel panel-primary">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 1
             <br>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 2
             <br>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 3
             <br>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             main
            </div>
            <div class="panel-body">
             x
            </div>
           </div>
          </td>
          <td style="vertical-align: bottom;">
           <div class="panel panel-primary">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 0
             <br>
              <span class="return-value">
               returns 1
              </span>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 1
             <br>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 2
             <br>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 3
             <br>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             main
            </div>
            <div class="panel-body">
             x
            </div>
           </div>
          </td>
          <td style="vertical-align: bottom;">
           <div class="panel panel-primary">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 1
             <br>
              <span class="return-value">
               returns 1
              </span>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 2
             <br>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 3
             <br>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             main
            </div>
            <div class="panel-body">
             x
            </div>
           </div>
          </td>
          <td style="vertical-align: bottom;">
           <div class="panel panel-primary">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 2
             <br>
              <span class="return-value">
               returns 2
              </span>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 3
             <br>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             main
            </div>
            <div class="panel-body">
             x
            </div>
           </div>
          </td>
          <td style="vertical-align: bottom;">
           <div class="panel panel-primary">
            <div class="panel-heading">
             factorial
            </div>
            <div class="panel-body">
             n = 3
             <br>
              <span class="return-value">
               returns 6
              </span>
             <br>
            </div>
           </div>
           <div class="panel panel-info">
            <div class="panel-heading">
             main
            </div>
            <div class="panel-body">
             x
            </div>
           </div>
          </td>
          <td style="vertical-align: bottom;">
           <div class="panel panel-primary">
            <div class="panel-heading">
             main
            </div>
            <div class="panel-body">
             x = 6
            </div>
           </div>
          </td>
         </tr>
        </tbody>
       </table>
      </div>

In the diagram, we can see how the stack grows:

1. `main` calls `factorial`
2. `factorial` then calls itself , until `factorial(0)` does not make a recursive call
3. the call stack unwinds, each call to factorial returning its answer to the caller, until `factorial(3)` returns to main

Here’s an [interactive visualization of factorial](http://www.pythontutor.com/visualize.html#mode=display). 
You can step through the computation to see the recursion in action. 

## TBD..
