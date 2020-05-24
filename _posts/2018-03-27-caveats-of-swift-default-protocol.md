---
layout: post
title: Caveats of Swift default protocol extensions
date: '2018-03-27T20:14:00.001+01:00'
author: luciano_marisi
tags:
- iOS
- swift
- debugging
permalink: /2018/03/caveats-of-swift-default-protocol.html
---

**TL;DR - Think carefully whether or not you need to add a default implementation to a property defined on a protocol**

Be prepared to avoid wasting a day of painful debugging caused by an obscure scenario in Swift! Swift 2 introduced the ability to add [extensions to protocols](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Extensions.html), for example: 

```swift
protocol SomeProtocol {
	var property: String { get }
}

extension SomeProtocol {
	var property: String {
		return "Default value"
	}
}
```

This is a great language feature but it introduces the potential for subtle bugs when modifying the protocol. In this post I'll cover 2 cases: renaming a property, and changing the type of a property.

## Case 1: Renaming a property defined in a protocol

Protocols can define properties to be implemented by the conforming type. For example, an http request can be modelled by the following:

```swift
enum HTTPMethod {
	case get
	case post
}


protocol HTTPRequest {
	var httpMethod: HTTPMethod { get }
}
```

In the majority of cases the http method used by iOS apps will be a GET request, so it's sensible to define that _all_ http requests default to `.get`:

```swift
extension HTTPRequest {
	var httpMethod: HTTPMethod {
		return .get
	}
}
```

A POST http request could be defined like this:

```swift
struct ExampleHTTPRequest: HTTPRequest {
	var httpMethod: HTTPMethod = .post
}
```

An `HTTPClient` can then _make_ http requests using the `httpMethod` defined in the `HTTPRequest` `protocol`:

```swift
struct HTTPClient {
	func make(httpRequest: HTTPRequest) {
		// some implementation using the http method
	}
}

let client = HTTPClient()
client.make(httpRequest: ExampleHTTPRequest()) -> uses a POST
```

Imagine that some time later the `httpMethod` gets renamed to `method`:

```swift
protocol HTTPRequest {
	var method: HTTPMethod { get }
}

extension HTTPRequest {
	var method: HTTPMethod {
		return .get
	}
}
```

After this change, when the http client makes the `HTTPRequest` for the original `ExampleHTTPRequest` the `HTTPMethod` is now `get` because it is using the default implementation of `HTTPRequest` defined in it's extension:

```swift
let client = HTTPClient()
client.make(httpRequest: ExampleHTTPRequest()) -> uses a GET!
```

Note that the `ExampleHTTPRequest` didn't change but the behaviour when used in the `HTTPClient` is different. The `ExampleHTTPRequest` still defines `var httpMethod: HTTPMethod = .post` making it hard and confusing to debug.

## Case 2: Changing the type of a property defined in a protocol

In this second case instead of renaming the property we'll change the type. Let's start by telling the story of `JohnDoe` and `JaneDoe`, 2 devoted athletes. The first thing they learnt as a `Person` was to `Walk`:

```swift
protocol Walking {}

protocol Person {
	var moveAction: Walking? { get }
}

struct Walk: Walking {}
```

Since not every `Person` can walk, the `moveAction` is optional with a `nil` default:

```swift
extension Person {
	var moveAction: Walking? {
		return nil
	}
}
```

Once `JohnDoe` and `JaneDoe` learned how to `Walk` they looked like this:

```swift
struct JohnDoe: Person {
	var moveAction: Walking? = Walk()
}

struct JaneDoe: Person {
	var moveAction: Walking? = Walk()
}
```

After learning how to `Walk` they figured it was time to have a `Race`:

```swift
struct Race {
	var people: [Person]

	func start() {
		people.forEach { person in
			// use person's moveAction
		}
	}
}

let firstRace = Race(people: [JohnDoe(), JaneDoe()])
firstRace.start()
```

Both of them managed to `Walk` in this `Race`:

```swift
firstRace.people[0].moveAction // returns Walk
firstRace.people[1].moveAction // returns Walk
```

After `JohnDoe` won the `Race` by a great distance, the next step was to learn how to _run_:

```swift
protocol Running {}
```

A `Person` could now _run_:

```swift
protocol Person {
	var moveAction: Running? { get }
}

extension Person {
	var moveAction: Running? {
		return nil
	}
}
```

Lazy `JohnDoe` was confident that his `Walk` was good enough to win the next `Race` and didn't learn how to `Run`. However, `JaneDoe` who was much more clever than `JohnDoe` (and knew a thing or two about Swift) updated her `moveAction` to include `Running` so she could `Run`:

```swift
struct JaneDoe: Person {
	var moveAction: Running? = Run()
}
```

The 2nd `Race` was on! The whistle went and the race started:

```swift
let secondRace = Race(people: [JohnDoe(), JaneDoe()])
secondRace.start()
```

To `JohnDoe`s surprise he had suddenly forgotten how to `Walk` as a `Person` whereas `JaneDoe` `Run` like the wind:

```swift
secondRace.people[0].moveAction // returns nil for JohnDoe
secondRace.people[1].moveAction // returns Run for JaneDoe
```

`JohnDoe` couldn't move and lost the race without knowing what hit him. There wasn't even a compiler error/warning to tell him what happened. Some people have gone as far as saying he couldn't take the pressure and choked...

However, a careful inspection of the Swift code tells us that poor `JohnDoe` was stripped of his ability to `Walk` as a `Person` when the `Person` protocol's `moveAction` was modified to `Running`. This is due to the fact that our default extension is now returning `nil` for `JohnDoe` when he is considered a `Person`. When accessing a property the reference type of the variable matters. When the instance of `JohnDoe` is considered a `Person`, the `moveAction` property from the `Person` default extension is used, i.e. `Running?`. Whereas when the instance is refered as `JohnDoe` the property used is the one defined by `JohnDoe`, i.e. `Walking?`:

```swift
(secondRace.people[0] as Person).moveAction // returns nil
(secondRace.people[0] as! JohnDoe).moveAction // returns Walk
```

The type for `moveAction` defined by `JohnDoe` is `Walking?` compared to `Running?` for `Person`. This subtle changes means `JohnDoe` no longer can `Walk` as `Person`


## Conclusions

If you're thinking that deprecating the old property is the way to go, it does not work. Deprecations don't have any effect whatsoever on the conforming type as that type can define it's own properties overriding the protocol ones:

```swift
protocol ExampleProtocol {
	@available(*, deprecated, renamed: "newProperty")
	var oldProperty: String { get }

	var newProperty: String { get }
}

extension ExampleProtocol {
	@available(*, deprecated, renamed: "newProperty")
	var oldProperty: String {
		return "Default value"
	}

	var newProperty: String {
		return "Default value"
	}
}
```

The following `ExampleStruct` has no warning or errors:

```swift
struct ExampleStruct: ExampleProtocol {
	var oldProperty: String
}
```

Neither changing it's type produces an error/warning:

```swift
struct ExampleStruct: ExampleProtocol {
	var oldProperty: Int
}
```

Therefore, I suggest that you think carefully when adding default implementations in extensions. If you absolutely must change a property that has default defined I suggest 2 approaches: 

1. Have complete confidence that you can propagate the change to every consuming type of your protocol
2. Remove the default implementation to force all consumers to implement it

This is specially important when modifying `public` interfaces since it's likely that you'll have no visibility of the projects that make use of that protocol. All in all, these solutions are not great and it's probably best to avoid having `public` extensions adding defaults altogether.

*I’d like to thank [Nahuel Marisi](http://twitter.com/nmarisi), [Daniel Haight](https://twitter.com/Daniel1of1) and [Neil Horton](http://twitter.com/neil3079) for reviewing this article.*