---
layout: post
title: Extending Swift generic types
date: '2016-03-16T23:32:00.001Z'
author: luciano_marisi
tags:
- swift
permalink: /2016/03/extending-swift-generic-types.html
---

**TL;DR - Generic Swift types such as `Array` or `Dictionary` can be easily extended to provide methods to specific types by making their associated type(s) either conform to a protocol or inherit from a class**

Back in the days of Objective-C it was very common to extend `NSArray` using a category to add convenience methods. However, as far as I know since Objective-C did not have generics it was not possible to constrain these category method to a specific type of `NSArray`. In Swift, typed arrays are the norm and sometimes it's necessary and very useful to extend these typed arrays to suit own needs. This post looks at how can Swift generic types be extended.

## Extending an array of strings (i.e. `Array<String>`)

A few weeks ago I wanted to add a convenience method to an array of strings for a project I was working on. The first thing that came to my mind was to literally constrain the `Array` to a `String`. However, the moment I did this I got the following error: *Same-type requirement makes generic parameter 'Element' non-generic*.

<p align="center">
   <img src="/assets/images/extending_array_contrained_to_string.png"/>
</p>

Back to the drawing board then...

What this is error is saying is that we cannot make an associated type non-generic because Swift does not support this. Presumably, this is because it would defeat the purpose of having an associated type originally. This error can be reproduce by the following code:

```swift
protocol MyGenericProtocol {
  associatedtype MyGenericProtocolType  //typealias in swift 2.1 and below
}

struct MyGenericStruct<MyGenericStructType>: MyGenericProtocol {
  typealias MyGenericProtocolType = MyGenericStructType
}

extension MyGenericStruct where MyGenericStructType == String {} // compiler error
```

The only way to constrain an `extension` for `Array` is either through a protocol or class and not a struct. An associated type can be constrained by inheritance or conformance but not equality because that would mean it would cease to be an associated type.

#### Naturally we cannot inherit from a struct

Attempting to constrain an `Array` by inheriting from a struct would look something like this:

```swift
struct MyStruct {}

extension Array where Element: MyStruct {
  // Functions that should only work for an array of MyStruct
}
```

Annoyingly, attempting to do this throws a compiler error but it's only shown when used on an Xcode project and not a playground. Not even the line of code of the error is marked. This code produces this error: **Type 'Element' constrained to non-protocol type 'MyStruct'**.

![](facepalm.png)

Therefore, the following are valid cases to constrain an `Array`:

#### 1. By constraining the `Element` so that it conforms to a protocol:

```swift
protocol MyProtocol {}

extension Array where Element: MyProtocol {
  // Functions that should only work for an array of MyProtocol
}
```

#### 2.a. By constraining the `Element` so that it inherits from a class:

```swift
class MyClass {}

extension Array where Element: MyClass {
  // Functions that should only work for an array of MyClass
}
```

#### 2.b. The `Element` can also be constrained to an `Element` that inherits from a root `final class`

This is unusually valid, considering the original compiler error, *Same-type requirement makes generic parameter 'Element' non-generic*. Unless I'm missing something, by constraining an Array to a final class it means that the `Element` in the `Array` is no longer generic. Hence, maybe the error should say something about equality not being allowed:

```swift
final class MyRootFinalClass {}

extension Array where Element: MyRootFinalClass {
  // Functions that should only work for an array of MyRootFinalClass
}
```

#### 3. By constraining the `Element` so that it inherits from a class and conforms to a protocol 

```swift
class MyClass {}

protocol MyProtocol {}

extension Array where Element: MyClass, Element: MyProtocol {
  // Functions that should only work for an array of MyClass objects that conform to MyProtocol
}
```

### Practical example

Using what we learned, we can now solve the original problem. The first thing would be to create a custom protocol, `StringProtocol`, to be used to extend our `Array`. Within the protocol we need to declare the functions we need to use inside our extension. Finally, we need to declare that `String` conforms to `StringProtocol`.

```swift
protocol StringProtocol {
  func hasPrefix(prefix: String) -> Bool
}

extension String : StringProtocol {}

extension Array where Element: StringProtocol {
  
  func filterByPrefix(prefix: String) -> [Element] {
    return filter { (element) -> Bool in
      element.hasPrefix(prefix)
    }
  }
  
}

let strings = ["aa", "ab", "bc"]
strings.filterByPrefix("a") // ["aa", "ab"]
```

## Dictionary example

A similar and also highly useful scenario is about constraining a `Dictionary` for JSON decoding as it's the case of version 2.x of [JSONUtilities](https://github.com/lucianomarisi/JSONUtilities/blob/master/JSONUtilities/Dictionary%2BJSONKey.swift). With a simple extension we can use type inference to decode a JSON that has a `String` key to the correctly inferred type. For example, for decoding a `String` from a dictionary of weakly typed `AnyObjects`:

```swift
protocol StringProtocol {}
extension String : StringProtocol {}

// Error enum for handling missing mandatory keys
enum Error : ErrorType {
  case NoValueForKey(StringProtocol)
}

public protocol DecodingSupportedType {}
extension String : DecodingSupportedType {}

extension Dictionary where Key: StringProtocol {
  
  func decodingKey<ReturnType : DecodingSupportedType>(key: Key) throws -> ReturnType {
    
    guard let value = self[key] as? ReturnType else {
      throw Error.NoValueForKey(key)
    }
    return value
  }
  
  func decodingKey<ReturnType : DecodingSupportedType>(key: Key) -> ReturnType? {
    return self[key] as? ReturnType
  }
}
```

This `Dictionary` extension could then be easily using to decode a `[String: AnyObject]` while keeping the type safety.

```swift
let dictionary: [String: AnyObject] = ["key": "value"]

/// Type inference from variable type
let mandatoryValue: String = try! dictionary.decodingKey("key") //"value\n"
let optionalValue: String? = try? dictionary.decodingKey("key") //"Optional("value")\n"

// Handling failures with mandatory keys
do {
  let mandatoryValue: String = try dictionary.decodingKey("otherkey")
} catch let error {
  print(error) // "NoValueForKey("otherkey")\n"
}
```

sdas

![](like_a_boss.jpg)


## Conclusion

It's certainly not immediately obvious how to extend an `Array`, a `Dictionary` or any generic type for that matter. However, by understanding these simple concepts it's possible to create highly reusable and strongly type functions that only work for the type you expect.