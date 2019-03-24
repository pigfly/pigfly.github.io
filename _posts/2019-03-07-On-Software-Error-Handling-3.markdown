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

There are pros and cons for these two errors type, let's take a look at the pros first.

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
