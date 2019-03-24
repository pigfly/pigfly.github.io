---
layout: post
title: "On Software Exception Handling 1"
date: 2018-04-11 07:33:33 +1000
comments: true
header-img: "img/post-bg-10.jpg"
categories: [software-engineering, exception-handling]
reading_time: "10 mins"
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

Although, having to always define a new `Error` type for each validation process might again generate unnecessary boilerplate 
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
) throws {
    guard condition() else {
        let message = messageClosure()
        throw ValidationError(message: message)
    }
}
``` 

## Put it together

With the above in place, we can now implement all of our validation logic as dedicated validators - 
constructed using computed static properties on the Validator type. 

For example, here's how we might implement a validator for credentials:

```swift
extension Validator where Value == String {
    static var password: Validator {
        return Validator { string in
            try validate(
                string.count >= 7,
                errorMessage: "Password must contain min 7 characters"
            )

            try validate(
                string.lowercased() != string,
                errorMessage: "Password must contain an uppercased character"
            )

            try validate(
                string.uppercased() != string,
                errorMessage: "Password must contain a lowercased character"
            )
        }
    }
}
```
Since we marked the above function with throws, we’re now required to prefix any call to it with the `try` keyword — 
which in turn forces us to handle any errors thrown from it (or to convert its return value into an optional using `try?`).

For example, here we’re using our function to get user information on the button clicked, 
if the validation passed (no error was thrown), then we’ll continue by submitting that username to local storage service. 
— otherwise, we display the error that was encountered using a UILabel:

```swift
func onUserClickedButton(_ credentials: Credentials) {
    do {
        try getUserInfo(with credentials: credentials)
    } catch {
        errorLabel.text = error.localizedDescription
    }
}
```

## Human readable error

However, if we run the above code with an invalid username as input (such as “alex-jiang”), we’ll end up with a quite obscure 
error message displayed in our errorLabel:

```text
The operation couldn’t be completed. (App.ValidationError error 2.)
```

That's not great, perhaps it's even more confusing. 

We do not know what to do if we see this message, even worse, we’re 
exposing implementation details (such as the name of the error type) to the user.

Fortunately, we can borrow the power from `localized`. We just need to let our `ValidationError` conform to `LocalizedError`.

By doing that, and implementing its `errorDescription` property — we can now return an appropriate, localized message for each error case:

```swift
extension ValidationError: LocalizedError {
    var errorDescription: String? {
        switch self {
        case .lengthTooShort:
            return NSLocalizedString(
                "Your username needs to be at least 4 characters long",
                comment: ""
            )
        case .lengthTooLong:
            return NSLocalizedString(
                "Your username can't be longer than 14 characters",
                comment: ""
            )
        case .invalidCharacter(let character):
            let format = NSLocalizedString(
                "Your username can't contain the character '%@'",
                comment: ""
            )

            return String(format: format, String(character))
        }
    }
}
```

With the above change in place, our validation error from before will now get displayed in a much more user-friendly way:

```text
Your username can't contain the character '-'
```