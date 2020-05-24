---
layout: post
title: Using Swift 4's Codable protocol to configure your UI test environment
date: '2017-11-26T21:36:00.000Z'
author: luciano_marisi
tags:
- swift
- codable
- ui tests
permalink: /2017/11/using-swift-4-codable-protocol-for-ui-tests.html
---

**TL;DR - Take advantage of the new Swift 4 Codable protocol to encode the launch environment of your UI tests**

Native UI testing in iOS does not allow direct interaction from the UI test to the app target. In other words, the test code cannot change the state of the app code while it is running. 

XCUITests allow for information to be sent to the app by either the `launchArguments: [String]` or the `launchEnvironment: [String : String]`. These 2 properties can be read from the `ProcessInfo`. Historically, I've used the `launchEnvironment` to send specific flags, for example `["UITest-has-completed-onboarding": "true"]`. However, this can get messy as more options are required and a lot of code is needed to be able to send more complex data. 

The introduction of `Codable` in Swift 4 provides a much simpler approach for this kind of communication. In essence, a type that represents the launch environment can be encoded and decoded with very little effort thanks to `Codable`.

## The example

Define a `UITestEnvironment` type that conforms to `Codable` with any relevant data for your UI test environment. For example:

**Note: This type has to be included in _both_ the app target and the test target.**

```swift
struct UITestEnvironment: Codable {

	static let Key = "UITestEnvironmentKey"

	let hasCompletedOnboarding: Bool

	// Any other keys or data you'd like
}
```

Add a function to encode an `Encodable` type into a base64 JSON string to your test target:

```swift
extension Encodable {
	func base64EncodedJSONString() -> String? {
		guard let encodedData = try? JSONEncoder().encode(self) else { return nil }
		return encodedData.base64EncodedString()
	}
}
```

Add a function to decode a `Decodable` type from a base64 JSON string to your app target:

```swift
extension Decodable {
	static func decode(from base64EncodedJSONString: String) -> Self? {
		guard let jsonData = Data(base64Encoded: base64EncodedJSONString) else { return nil }
		return try? JSONDecoder().decode(self, from: jsonData)
	}
}
```

On your UI tests create the environment you need for that test, encode it and pass to the `launchEnvironment`:

```swift
let originalArguments = UITestEnvironment(hasCompletedOnboarding: true)
let encodedArguments = originalArguments.base64EncodedJSONString()!

let app = XCUIApplication()
app.launchEnvironment = [UITestEnvironment.Key: encodedArguments]
app.launch()
```

Read the launch environment from `ProcessInfo` and decode the data into `UITestEnvironment` on the app target where it is relevant to alter the state of the running app:

```swift 
if let encodedEnvironment = ProcessInfo.processInfo.environment[UITestEnvironment.Key],
	let decodedEnvironment = UITestEnvironment.decode(from: encodedEnvironment) {
	// Do something with the test environment
}
```

That's it! This is a very simple way to transmit information between your UI tests and your app. Since the only requirement for a UI test environment is that it conforms to `Codable` you may have multiple types for different tests instead of one `UITestEnvironment`.

*I’d like to thank [Nahuel Marisi](http://twitter.com/nmarisi) for reviewing this article.*