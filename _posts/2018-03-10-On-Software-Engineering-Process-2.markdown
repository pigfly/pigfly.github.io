---
layout: post
title: "On Software Engineering Process 2"
date: 2018-03-10 14:20:11 +1000
comments: true
header-img: "img/post-bg-08.jpg"
categories: [software-engineering, software-design]
reading_time: "15 mins"
---

These series are the reflection and study when I was leading an engineering team for all the aspects of software development.
In particular, this post covers how to handle *exception* cases in a way that is safe from bugs.

<!--more-->

In your career time, you've probably already seen various exceptions in different scenarios. For example, in Java, such as
[ArrayIndex­OutOfBounds­Exception](https://docs.oracle.com/javase/8/docs/api/?java/lang/ArrayIndexOutOfBoundsException.html),
[Null­Pointer­Exception](https://docs.oracle.com/javase/8/docs/api/?java/lang/NullPointerException.html). These exceptions generally
indicate our software run into a state that could cause problem, and the and the information displayed by the language when the 
exception is thrown can help you find and fix the problem.

## Exceptions for special results

> Exceptions can be used to improve the structure of code that involves procedures with special results.

An unfortunately common way to handle special results is to return special values.

For example, lookup operations in the Java library are often designed like this: you get an index of -1 when expecting a positive integer, 
or a null reference when expecting an object. This approach is OK if used sparingly, but it has two problems. 
First, it’s tedious to check the return value. 
Second, it’s easy to forget to do it. (We’ll see that by using exceptions you can get help from the compiler in this.)

Also, it’s not always easy to find a ‘special value’. Suppose we have a Facebook class with a lookup method. 
Here’s one possible method signature:

```swift
final class Facebook {
    func lookup(name: String) -> Person?
}
```

What should the method do if the facebook doesn’t have an entry for the person whose name is given? 
Well, we could return some special date that is not going to be used as a real date. Bad programmers have been doing this for decades; 
they would return 9/9/1970. 

Or in Swift, they could return optional to indicate the a person with particular name could not be found.
But its not obvious and robust enough, what if there are multiple errors happening during the lookup ? They will would not be able
to identify which error it might occur, all we have is the optional now...

### A better approach

The function throws an exception:

```swift
final class Facebook {
    func lookup(name: String) throws -> Person? {
    ...
    if ...not found...
        throw LookupException.notFound(name: name)
    ...
    }
}
```

and the caller handles the exception with a catch clause. For example:

```swift
let book: Facebook = ...
try {
    let person = book.lookup("Alex")
} catch (LookupException.notFound) {
    // his name was not in the face book
}
```

Now there’s no need for any special value, nor the checking associated with it.


## Exception design considerations

The rule we have given — use exceptions for special results (i.e., anticipated situations), and to signal bugs 
(unexpected failures)— makes sense, but it isn’t the end of the story.

