---
layout: post
title: Workaround for serializing Codable fragments
date: '2018-07-20T10:54:00.001+01:00'
author: luciano_marisi
tags:
- iOS
- swift
- codable
permalink: /2018/07/workaround-for-serializing-codable-fragments.html
---

**TL;DR - Wrap the `Codable` type in an array and use a `JSONDecoder` to convert it to `Data`**

Serialization of data has been immensely improved by the introduction of `Codable`. This interface provides an api with much less boilerplate than the classic `NSCoding`.

Currently Swift provides decoders and encoders for 2 types of data, JSON and property list:

- [JSONDecoder](https://developer.apple.com/documentation/foundation/jsondecoder) & [JSONEncoder](https://developer.apple.com/documentation/foundation/jsonencoder) 
- [PropertyListDecoder](https://developer.apple.com/documentation/foundation/propertylistdecoder) & [PropertyListEncoder](https://developer.apple.com/documentation/foundation/propertylistencoder)

One of the limitations is that neither of them support single values, also known as fragments. For example:

```swift
let json = "A string".data(using: .utf8)!
do {
	try JSONDecoder().decode(String.self, from: json)
} catch {
	// Error Domain=NSCocoaErrorDomain Code=3840 "JSON text did not start
	// with array or object and option to allow fragments not set."
	// UserInfo={NSDebugDescription=JSON text did not start with array
	// or object and option to allow fragments not set.}
}
```

The issue related to JSON is raised on the Swift JIRA, [SR-6163](https://bugs.swift.org/browse/SR-6163). The reason this happens is because [JSONDecoder is implemented using JSONSerialization](https://github.com/apple/swift/blob/master/stdlib/public/SDK/Foundation/JSONEncoder.swift#L232) but it doesn't provide an option to allow fragments which is where the error comes from.

## Real life example

A popular tool for persistency in Swift is [Disk](https://github.com/saoudrizwan/Disk). One of it's features is the ability to save a `Codable` type to disk:

```swift
public extension Disk {
	static func save<T: Encodable>(_ value: T, to directory: Directory, as path: String) throws {
		// Implementation using JSONEncoder
	}
}
```

The library is great but it's limited by the Swift issue, if we try to save a string it'll fail:

```swift
try Disk.save("A string", to: .documents, as: "filename.extension") // fails
```

No compiler errors are given because `String` is `Encodable` but this will throw the following error:

> Error Domain=NSCocoaErrorDomain Code=4866 "Top-level String encoded as string JSON fragment." UserInfo={NSCodingPath=(
), NSDebugDescription=Top-level String encoded as string JSON fragment.}

Both the API and the [documentation](https://github.com/saoudrizwan/Disk#usage) (_Disk currently supports persistence of the following types: Codable, ..._) suggest this _should_ work. However, since Disk is implemented using `JSONEncoder` for serialization it doesn't.

_Note that I used Disk as an example because it's well documented, has good tests and works very nicely. The fact that I'm pointing a bug doesn't mean it's anything other than great._

## One possible (and temporary) solution to this problem

Since Swift does not support fragments a reliable and somewhat questionable implementation is to wrap the `Encodable ` type into an array to guarantee it'll encode:

```swift
private extension Encodable {
	func encode() -> Data? {
		return try? JSONEncoder().encode([self])
	}
}
```

This serialized `Data` can then be stored into the disk or sent somewhere.

To retrieve the original `Codable` type we can revert the process. We decode the data, take the first item in the array and cast it to the corresponding type. Note that this returns a generic parameter because there is no easy way to encode the original type.

```swift
extension Data {
	func decode<T: Decodable>() -> T? {
		return (try? JSONDecoder().decode([T].self, from: self))?.first
	}
}
```

Going back to the original example from [SR-6163](https://bugs.swift.org/browse/SR-6163), these extensions can be used like this:

```swift
let jsonPrimitive = "A string"
let encodedData = jsonPrimitive.encode()
let decodedValue: String? = encodedData?.decode() // "A string"
```

## Conclusions

The proposed workaround is by no means elegant or space efficient but it reliably converts any `Codable` type to `Data` and back to `Codable`. Therefore, this is good tool to have in your toolbelt until swift provides a decoder/encoder that allows fragments as `Codable` provides a very convenient way of serializing objects.

*I’d like to thank [Nahuel Marisi](http://twitter.com/nmarisi), [Daniel Haight](https://twitter.com/Daniel1of1) and [Neil Horton](http://twitter.com/neil3079) for reviewing this article.*