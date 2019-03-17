---
layout: post
title: "On Software Engineering Process 1"
date: 2018-03-07 23:39:49 +1000
comments: true
header-img: "img/post-bg-07.jpg"
categories: [software-engineering, software-design]
reading_time: "15 mins"
---

These series are the reflection and study when I was leading an engineering team for all the aspects of software development.
In particular, this post covers how to write better specification for your code.

<!--more-->

As I spend most of time in the financial sector, seeing how those giants trying best to turn the ship from "waterfall" 
process into "agile" way, and some of these eventually developed own ways of working.
However, these is one crucial pattern that is in common for these financial groups: **risk-averse**. 

More often than not, I often saw people doing the risk-driven development without actually realising it.

## Risk-driven development

> Risk = Probability of (failure) * Cost of (failure) 

what do people usually do when dealing with failures ?

- list failures & determine their risks
- devise a strategy to reduce highest risks

What I haven seen some good examples of failures ?

- performance is unacceptable
- product is unusable because its too complex
- customer changes mind about what product does
- developer solves the wrong problem
- product fails in catastrophic way
- competitor beats you into marketplace
- product has reputation for bugs
- development runs out of time and money
- developers rely on platform that turns out bad 

First let's focus on the subcategory of all these failures that would be ignored most of the time: 
the engineering part, to write good specifications.

### Specification (a.k.a. Design Docs)

Most of the annoying bugs in our codebase arise due to misunderstandings about the behavior or intention between two pieces
of code. However, every engineer has specifications in mind, only few write them down. That leads to a very awkward position,
different engineers on the same team have *different* specifications in mind. When the code breaks, it's hard to determine
where the error is.

What makes this worse is, most of managers are blind to see the benefits of precise specifications in the code or in a form of documentation,
namely, to let you blame (to code fragments, not people!), and can spare you the pain of randomly try-and-error where a fix should go.

If you’re not convinced that reading a spec is easier than reading code, take a look at some of the standard Java specs and compare them to the source code that implements them.

| add                 |
|---------------------|
| `public BigInteger add(BigInteger val)` |
| Returns a BigInteger whose value is `(this + val)`    |
| **Parameters:** |
| `val` - value to be added to this BigInteger. |
| **Return:** |
| `this + val` |

Here is the source code from Java 8:

```java
if (val.signum == 0)
    return this;
if (signum == 0)
    return val;
if (val.signum == signum)
    return new BigInteger(add(mag, val.mag), signum);

int cmp = compareMagnitude(val);
if (cmp == 0)
    return ZERO;
int[] resultMag = (cmp > 0 ? subtract(mag, val.mag)
                   : subtract(val.mag, mag));
resultMag = trustedStripLeadingZeroInts(resultMag);

return new BigInteger(resultMag, cmp == signum ? 1 : -1);
```

The spec for `BigInteger.add` is straightforward for caller to understand, and if we have questions about corner cases, 
the `BigInteger` class provides additional human-readable documentation. If all we had was the code, 
we’d have to read through the `BigInteger` constructor, `compare­Magnitude` , `subtract` , and `trusted­StripLeadingZero­Ints` just as a starting point.


#### Code behaviors the same

Consider these two functions. Are they talking about the same thing ?

```swift
static func findFirst(arr: [Int], val: Int) -> Int {
    for (i, n) in arr.enumerated() where n == val {
        return i
    }
    return arr.length
}

static func findLast(arr: [Int], val: Int) -> Int {
    for (i, n) in arr.reversed().enumerated() where n == val {
        return i
    }
    return -1;
}
```

The answer is obviously no.
Not only do these functions have different code, they indeed have different behaviors:

- when `val` is missing, `findFirst` returns the length of `arr` and `findLast` returns -1
- when `val` appears twice, `findFirst` returns the lower index and `findLast` returns the higher
 
What does this tell us ?

The notion of two code snippets behavior the same is *depending on the caller*:

- when `val` occurs at exactly one index of the array, the two methods behave the same
    
So in order to refactor these two code snippet into one, or substitute one implementation for another, and to know when this is
acceptable, we need our specification to include what the caller depends on.

In that case, our specification might be like:

```java
static int find(int[] arr, int val)
  requires: val occurs exactly once in arr
  effects:  returns index i such that arr[i] = val
```


#### Specification structure

A specification of a function consists of several clauses:

- a *precondition* , indicated by the keyword `requires`
- a *postcondition* , indicated by the keyword `effects`
    
The precondition is an obligation on the caller. It’s a condition over the state in which the method is invoked.

The postcondition is an obligation on the implementer of the method. If the precondition holds for the invoking state, 
the method is obliged to obey the postcondition, by returning appropriate values, throwing specified exceptions, 
modifying or not modifying objects, and so on.


#### Specification in Swift

Some languages (e.g. [Eiffel](https://en.wikipedia.org/wiki/Eiffel_(programming_language)) ) put preconditions and postconditions as a fundamental part of the language, 
as expressions that the runtime system (or even the compiler) can automatically check to enforce the contracts between caller and implementers.

Swift does not go quite so far, although [precondition](https://developer.apple.com/documentation/swift/1540960-precondition) in Swift can
make necessary condition check for forward progress, it turns out to be an *non-recoverable* failure. So we need to put these specification
as part of comments and rely on human beings to check and guarantee it.

Fortunately, Swift has a [Markup](https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_markup_formatting_ref/index.html) 
format for documentation. So we could put preconditions into `Parameters`, and postconditions into `Returns` and `Throws`.
So a specification like this:

```swift
static func find(arr: [Int], val: Int) -> Int
  requires: val occurs exactly once in arr
  effects:  returns index i such that arr[i] = val
```

might be translated in Swift like this:

```swift
    /// Find a value in an array
    ///
    /// - Parameters:
    ///   - arr: array to search, requires that val occurs exactly once in arr
    ///   - val: value to search for
    /// - Returns: index i such that arr[i] = val
    static func find(arr: [Int], val: Int) -> Int
```
