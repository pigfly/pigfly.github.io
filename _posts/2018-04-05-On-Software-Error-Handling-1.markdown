---
layout: post
title: "On Software Exception Handling 1"
date: 2018-04-05 06:12:33 +1000
comments: true
header-img: "img/post-bg-09.jpg"
categories: [software-engineering, exception-handling]
reading_time: "15 mins"
---

These series are the reflection and study when I was leading an engineering team for all the aspects of software development.
In particular, this post covers the techniques for better exception handling in Swift❤️.

<!--more-->

We've touched the design principle for exception handling previously, now it's time to take a look at a few concrete examples
for how to do error handling in our day-to-day job. In particular, we'll use Swift as an example in this post.

## Optionals != error handling

Swift's [optionals](https://developer.apple.com/documentation/swift/optional) is great👍, it forces compiler to check value that
can be legitimately missing. However, it hides away the *underlying cause* to why error occurs in the first place.

Let's consider this example, here we have a function that tries to get user information from local storage. Before it hit the local storage,
it needs to validate the given credential is legit or not:

```swift
func getUserInfo(with credentials: Credentials) -> User? {
    guard credentials.username.count >= 4 else {
        return nil
    }

    guard credentials.password.count >= 10 else {
        return nil
    }

    // Additional validation
    ...

    localStorageService.getUserInfo(with: credentials) ...
}
``` 

The problem is there are multiple cases in the runtime that could go wrong, but we only use `nil` value to deal with it,
which hides the source of true error in the first place.

If you are an indie developer, it's probably fine as you would be the solo player in your codebase. However, if you are one of those
developers work in a large team, then this approach won't be favoured...

## Mark function throws

We'll start by defining an enum containing cases for each error that can occur within our validation code - looking something like this:

```swift
enum ValidationError: Error {
    case lengthTooShort
    case lengthTooLong
    case invalidCharacter(Character)
    ...
}
```

Using the above error enum, we’ll mark our function as being able to throw errors using throws, 
and use the `throw` keyword to trigger an error in case a validation requirement doesn't meet — like this:

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