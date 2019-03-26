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

- When the session request succeeds, the `Data?` will contain our response from the server, and the `Error?` is nil
- When the session request fails, the `Error?` will contain error information, and the `Data?` is nil

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

## E: Error Or E ?

The original [proposal](https://github.com/apple/swift-evolution/pull/757) for adding `Result<T>` to standard library is as follows:

```swift
enum Result<T> {
    case success(T)
    case failure(Error)
}
```

The original proposal only apply type constraint for success case, leaving the failure case unbound.

One would argue why not apply the same type constraint for failure case ?

Because the `Error` design in swift is nothing but a protocol, we do not need to specify the error type when throwing error:

```swift
func methodCanThrow() throws {
    if somethingGoesWrong { ... }
}

do {
    try methodCanThrow()
} catch {
    if error is SomeErrorType {
        // ...
    } else if error is AnotherErrorType {
        // ...
    }
}
```

However, if we have something like `Result<T, E: Error>`, then we need to specify a concrete error type for `E`. By doing this,
will violate the design principle behind the `Error`, since things like `Result<Response, Error>` is not allowed in swift.

## The pros for `Result<T, E: Error>`

There are pros and cons for these two errors type, let's take a look at the pros first. 

1. Rely on the compiler to lock down the error type:

```swift
enum UserRegisterError: Error {
    case duplicatedUsername
    case unsafePassword
}

userService.register("user", "password") {
    result: Result<User, UserRegisterError> in
    switch result {
    case .success(let user):
        print("User registered: \(user)")
    case .failure(let error):
        if error == .duplicatedUsername {
            // ...
        } else if error == .unsafePassword {
            // ...
        }
    }
}
```

As above, the error type has been identified as `UserRegisterError` in compiler, it would be trivial to do error handling in failure case.

2. Easy to extend `Result<T, E: Error>`

For example, some asynchronous operations may never fail, and for these operations, we don't have to use switch to check the branch.

A good example is `Timer`, the asynchronous operation will always guarantee to succeed, we could use a special type to address this:

```swift
enum NoError: Error {}

func run(after: TimeInterval, 
         done: @escaping (Result<Timer, NoError>) -> Void ) 
    {
        Timer.scheduledTimer(withTimeInterval: after, repeats: false) { timer in
            done(.success(timer))
        }
    }
}
```

we would suppose to write our code like this:

```swift
run(after: 2) { result in
    switch result {
    case .success(let timer):
        print(timer)
    case .failure:
        fatalError("Never happen")
    }
}
```

However, with the extension for `NoError`, things become much easier:

```swift
extension Result where E == NoError {
    var value: T {
        if case .success(let v) = self {
            return v
        }
        fatalError("Never happen")
    }
}

run(after: 2) {
    // $0.value is the timer object
    print($0.value)
}
```

## The cons for `Result<T, E: Error>`

The downside for this bounded Error type is all on the caller side:

1. Due to the historical reason, the Cocoa API is designed in such way that most of the errors have no type(e.g. `NSError`),
sometimes, you are forced to write something like `Result<SomeValue, NSError>`

2. If there are multiple level error cases, we would end up with nested error handling code

Here we have three levels of custom errors:

```swift
// Error type for user regisration
// Triggered when response succeed but the returned data indicates registration fails
enum UserRegisterError: Error {
    case duplicatedUsername
    case unsafePassword
}

// Error type for server API response
// Triggered when resquest succeed but the response status code is not within 200 class
enum APIResponseError: Error {
    case permissionDenied // 403
    case entryNotFound    // 404
    case serverDied       // 500
}

// Error type for APIClient
// Triggered when any error occurs during request and response cycle
enum APIClientError: Error {
    // request timeout
    case requestTimeout
    // resquest succeed，but response code is not 200
    case apiFailed(APIResponseError)
    // request succeed, response code is 200, but fails to parse the response data
    case invalidResponse(Data)
    // resquest and response both succeed, but fail to complete a meaningful action (e.g. registration fail..)
    case apiResultFailed(Error)
}}
```

The above `APIClientError` covers all possible error cases for a single API request, but it's not so pleasant for the caller to consume:

```swift
API.send(request) { result in
    switch result {
    case .success(let response): //...
    case .failure(let error):
        switch error {
        case .requestTimeout: print("Timeout!")
        case .apiFailed(let apiFailedError):
            switch apiFailedError: {
                case .permissionDenied: print("403")
                case .entryNotFound: print("404")
                case .serverDied: print("500")
            }
        case .invalidResponse(let data):
            print("Invalid response body data: \(data)")
        case .apiResultFailed(let apiResultError):
            if let apiResultError = apiResultError as? UserRegisterError {
                switch apiResultError {
                    case .duplicatedUsername: print("User already exists.")
                    case .unsafePassword: print("Password too simple.")
                }
            }
        }
    }
}
```

Believe me, the last thing you want is to write your code like this and let your teammates to review your code🤪...

However, you could provide a custom struct like `AnyError` to encapsulate the `Error` itself:

```swift
struct AnyError: Error {
    let error: Error
}
```

However, by using `Result<Value, AnyError>`, we lose all the benefits that `Result<T, E: Error>` bring to us.

## Part of Swift 5

A good news is the `Result<T>` becomes part of standard library in Swift 5🎉, so individual frameworks and apps no longer 
have to define their own — and more importantly, no longer have to convert between different flavors of the same kind of type.

There is also another proposal which comes along with `Result<T>`, that is `Error` protocol is *self-conforming*🤔.

That is, the above NSError-based technique is no longer necessary in Swift 5:

```swift
extension URLSession {
    func dataTask(
        with request: URLRequest, 
        completionHandler: @escaping (Result<(Data, URLResponse), Error>) -> Void) 
        -> URLSessionDataTask 
    {
        return dataTask(with: request) { data, response, error in
            if error != nil {
                completionHandler(.failure(error!))
            } else {
                completionHandler(.success((data!, response!)))
            }
        }
    }
}
```

Very neat! 😎

Also, Swift 5’s implementation of Result includes a get() method, that either returns the result’s value, or throws an error:

```swift
extension Result {
    func get() throws -> Value {
        switch self {
        case .success(let value):
            return value
        case .failure(let error):
            throw error
        }
    }
}
```

## Conclusion

Though a more sophisticated [proposal](https://github.com/apple/swift-evolution/blob/af284b519443d3d985f77cc366005ea908e2af59/proposals/0192-non-exhaustive-enums.md) 
has been put in place to address all these issues, error handling is a tricky balance between the language designer and the language consumer.
