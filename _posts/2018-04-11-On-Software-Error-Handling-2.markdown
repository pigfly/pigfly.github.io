---
layout: post
title: "On Software Exception Handling 2"
date: 2018-04-11 07:33:33 +1000
comments: true
header-img: "img/post-bg-10.jpg"
categories: [software-engineering, exception-handling]
reading_time: "20 mins"
---

These series are the reflection and study when I was leading an engineering team for all the aspects of software development.
Following on the previous post on error handling in Swift, in this post, we will have a look at how to improve🚀 our error handling
logic.

<!--more-->

Although Swift has a sophisticated and powerful type inference system, we can't always be sure that functions will 
get valid input - sometimes a runtime check is the only choice.

## Input Validation

Recap the code snippet in [previous example](https://pigfly.github.io/software-engineering/exception-handling/2018/04/05/On-Software-Error-Handling-1/),

```swift
func getUserInfo(with credentials: Credentials) throws -> User {
    guard credentials.username.count >= 4 else {
        throw ValidationError.lengthTooShort
    }

    guard credentials.password.count >= 10 else {
        throw ValidationError.lengthTooShort
    }
    
    for character in username {
        guard character.isLetter else {
            throw ValidationError.invalidCharacter(character)
        }
    }

    // Additional validation
    ...

    localStorageService.getUserInfo(with: credentials) ...
}
``` 

Even though we're only validating three pieces of data above, our validation logic can end up growing much quicker than we might expect.

So let's see if we can do some decoupling and also improve our control flow in the process.

## Validation Rule

Let's start by creating a generic struct for all validation logic. We'll call it *Validator* and make it a simple `struct`
that holds a validation rule for a given Value type:

```swift
struct Validator<Value> {
    let rule: (Value) throws -> Void
}
```

Using the above, we'll be able to create validators that throw an error whenever a value fails to pass validation.

## Too much throw ?

Although, having to always define a new `Error` type for each validation logic might again generate unnecessary boilerplate code
(especially if all we want to do with an error is to display it to the user) 

So,  let's also introduce a function that lets us write validation logic by simply passing a Bool condition and a 
message to display to the user in case of a failure:

```swift
struct ValidationError: LocalizedError {
    let errorMessage: String
    var errorDescription: String? { return errorMessage }
}

func validate(
    _ condition: @autoclosure () -> Bool,
    errorMessage messageClosure: @autoclosure () -> String
) throws
    {
    guard condition() else {
        let message = messageClosure()
        throw ValidationError(message: message)
    }
}
``` 

## Put it together

With the above in place, we can now implement all of our validation logic as different validators - 
constructed using computed static properties on the Validator type. 

For example, here's how we might implement a validator for credentials:

```swift
extension Validator where Value == String {
    static var credentials: Validator {
        return Validator { string in
            try validate(
                string.count >= 7,
                errorMessage: "credentials must contain min 7 characters"
            )

            try validate(
                string.lowercased() != string,
                errorMessage: "credentials must contain an uppercased character"
            )

            try validate(
                string.uppercased() != string,
                errorMessage: "credentials must contain a lowercased character"
            )
        }
    }
    
    static var password: Validator {
        ...
    }
}
```

As above, we create a validator type with a rule that contains three different pieces of validation logic.

If any of the three validation logic fails, it will throw a `ValidationError` with the message to indicate why it will fail.

## One step further

let's create another validate overload that'll act as a bit of *syntactic sugar*, by letting us call it with the value 
we wish to validate and the validator to use:

```swift
func validate<T>(_ value: T,
                 using validator: Validator<T>) throws {
    try validator.rule(value)
}
```

The above will let us make our code requiring input validation very nice and clean:

```swift
func getUserInfo(with credentials: Credentials) throws -> User {
    try validate(credentials.username, using: .credentials)
    try validate(credentials.password, using: .password)
    
    // Additional validation
    ...

    localStorageService.getUserInfo(with: credentials) ...
}
```

Perhaps even better, is that we can now deal with all validation errors in a single place, and then simply display the 
`localized` description of any thrown error to the user:

```swift
do {
    try getUserInfo(with: credentials)
} catch {
    errorLabel.text = error.localizedDescription
}
```
