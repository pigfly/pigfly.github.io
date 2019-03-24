---
layout: post
title: "On Software Exception Handling 3"
date: 2019-03-07 23:22:02 +1000
comments: true
header-img: "img/post-bg-11.jpg"
categories: [software-engineering, exception-handling]
reading_time: "20 mins"
---

These series are the reflection and study when I was leading an engineering team for all the aspects of software development.
In this post, we will explore an interesting yet an important concept in Swift: `Result<T>` 🎉

<!--more-->

Personally, I've been using `Result<T>` in my own projects for years, also it has been introduced into the standard library
as part of Swift 5. Before we jump into the evolution of `Result<T>` in swift community, let's rewind our memory back a bit,
to see what would happen without this Result type.

## Background

When performing networking requests, most of the requests are asynchronous by default, for example:

```swift
URLSession.shared.dataTask(with: request) {
    data, response, error in
        if error != nil {
            handle(error: error!)
        } else {
            handle(data: data!)
        }
}
``` 

Here we have three parameters `(Data?, URLResponse?, Error?)`, and all of these are optional.

- When the session request succeeds, the `Data?` will contain our response from the server, `Error?` is nil
- When the session request fails, the `Error?` will contain error information, `Data?` is nil

As a matter of fact, the `Error?` and `Data?` value would be mutual exclusive, there is no way that both `Data?` and `Error?`
will be nil or contain value.

However, there's no compile-time guarantee that the data we're looking for is actually there☹️.

## Result<T>

We could avoid this dilemma by introducing `Result<T>`.

The idea behind this is so simple, by turing each result into two separate states, with the help of enum containing 
a case for each state — one for *success* and one for *failure*:

```swift
enum Result<T, E: Error> {
    case success(T)
    case failure(E)
}
```

Using the above, we'll be able to apply this idea into the previous network request:

```swift
extension URLSession {
    func dataTask(
        with request: URLRequest, 
        completionHandler: @escaping (Result<(Data, URLResponse), NSError>) -> Void) 
        -> URLSessionDataTask 
    {
        return dataTask(with: request) { data, response, error in
            if error != nil {
                completionHandler(.failure(error! as NSError))
            } else {
                completionHandler(.success((data!, response!)))
            }
        }
    }
}


URLSession.shared.dataTask(with: request) { result in
    switch result {
    case .success(let (data, _)):
        handle(data: data)
    case .failure(let error):
        handle(error: error)
    }
}
```

Bingo, we can now avoid checking the optional value and let the compiler guarantee that there are non-optional values 
​​in the corresponding case branch.

Also, this design concept has been adopted in [Swift Package Manager](https://github.com/apple/swift-package-manager/blob/master/Sources/Basic/Result.swift)
and [Alamofire](https://github.com/Alamofire/Alamofire/blob/master/Source/Result.swift)

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
