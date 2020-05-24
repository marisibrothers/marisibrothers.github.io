---
layout: post
title: The Yin and Yang of Unit Testing in iOS
date: '2017-01-31T20:37:00.002Z'
author: luciano_marisi
tags: 
permalink: /2017/01/the-yin-and-yang-of-unit-testing-in-ios.html
---

**TL;DR - Sometimes it's more suitable to test the interface and other times the implementation of a system. Understanding the difference between the two will help write more valuable tests as sometimes the approaches are complementary.**

When writing a unit test the approach would depend on how much you know about the code and how much you would like your test to know about the code your testing. Arguably, the key difference between black box and white box testing is that in the former the interface is being tested whereas in the latter the implementation is being tested. I'm going to focus on unit tests in the context of iOS even though these concepts are relevant for other types of tests such integration and system tests.

*Note: examples written in Swift 3.0*

## Black box testing

As the following diagram shows, in black box testing the goal is to test that for a given input an expected output is returned. The implementation of the blackbox may not be known in this case.

<p align="center">
   <img src="/assets/images/black_box_testing.png" width="50%" />
</p>

For example, when using a closed source 3rd party library this may be a suitable way to test code that uses it. Alternatively, if you expect your code to work irrespective of the implementation details you could treat it as a black box.

I've been implementing SSL public key pinning recently so I've used a real life example related to this. Let's say we have a public function used to retrieve the public keys from all the certificates in the app's bundle. The code would probably look something like this:

```swift
public struct TrustStore {

  public static func publicKeys(in bundle: Bundle = Bundle.main) -> [SecKey] {
    let localCertificates = certificates(in: bundle)
    return localCertificates.flatMap { $0.publicKey() }
  }
  
  private static func certificates(in bundle: Bundle = Bundle.main) -> [SecCertificate] {
    let certificateExtensions = Set([".cer", ".CER", ".crt", ".CRT", ".der", ".DER"])
    let paths = certificateExtensions.map { fileExtension in
      bundle.paths(forResourcesOfType: fileExtension, inDirectory: nil)
      }.joined()

    return paths.flatMap {
      guard let certificateData = try? Data(contentsOf: URL(fileURLWithPath: $0)) as CFData else {
          return nil
      }
      return SecCertificateCreateWithData(nil, certificateData)
    }
  }
  
}

extension SecCertificate {
  func publicKey() -> SecKey? {
    var publicKey: SecKey?
    
    let policy = SecPolicyCreateBasicX509()
    var trust: SecTrust?
    let trustCreationStatus = SecTrustCreateWithCertificates(self, policy, &trust)
    
    if let trust = trust, trustCreationStatus == errSecSuccess {
      publicKey = SecTrustCopyPublicKey(trust)
    }
    
    return publicKey
  }
}
```

Most of the code wraps C functions from the [Security](https://developer.apple.com/reference/security) framework. We could test that these functions are called in specific ways. However, even though that would be a valid test I think that given the number of framework functions involved in getting to the result it's easier to think about the code as a whole blackbox system. For such a test we could add a few certificates with known public keys to the test bundle and write a unit test asserting that `func publicKeys(in bundle: Bundle = Bundle.main) -> [SecKey]` returns the expected public keys. 

Assuming that only one public key exists in our test bundle, our unit test would look something like the following:

```swift
func testCorrectPublicKeyExist() {
  let firstPublicKey = TrustStore.publicKeys().first!
  let expectedPublicKey = // Your public key
  XCTAssertEqual(firstPublicKey, expectedPublicKey)
}
```
*Note that the real test is actual slightly more complex but for the sake of clarity it has been simplified.*

Essentially we are testing that the contract of the API we created actually does what we expect and we are using the Security framework correctly. If in the future the implementation would use something other than this framework our tests would still pass. In this case the value of testing this function is whether the output is correct according to the documented interface. There is much less value to check whether the correct C functions were called.

This has the advantage that the tests help document the written code. By testing the interface they demonstrate what was the intention of the programmer when building the system. Effectively documenting the code in a way that is less likely to go out of date as it's code that gets run frequently. Also tests can be written before the implementation (i.e. TDD) meaning that the problem can be thought more throughly in a wider context.

On the other hand, since the internals of a system are not known it's unlikely that all code paths and edge cases would be covered. It's relatively straight forward to test the happy paths but very hard to test all the possible failure cases.

## White box testing

White box testing is the opposite of black box testing. In this case the internals of the system are known. Therefore, tests can be written with the implementation in mind. This is very common in unit testing as it allows to test all the paths with a unit of code.

Sometimes when working with `(NS)Operation`s it's necessary to cancel all operations before adding new ones to the queue. For example, the `doSomeWork` function does exactly this:

```swift
class SomeObject {
  
  let operationQueue: OperationQueue
  
  init(operationQueue: OperationQueue = OperationQueue()) {
    self.operationQueue = operationQueue
  }
  
  func doSomeWork() {
    operationQueue.cancelAllOperations()
    // More stuff
  }
}
```

To be able to test this side effect we need to know the implementation of `doSomeWork`. The `OperationQueue` is one of the parameters in the initializer so that we can mock in our tests. As an aside, this method is called [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection), and in this example we are using constructor injection. The mocked `OperationQueue` looks something like this:

```swift
class MockOperationQueue: OperationQueue {
  
  var cancelAllOperationsCallCount = 0
  
  override func cancelAllOperations() {
    super.cancelAllOperations()
    cancelAllOperationsCallCount += 1
  }
}
```

We can then use this mock in our testcase to assert that calling `doSomeWork` actually calls `cancelAllOperations`:

```swift
class TestCase: XCTestCase {
  
  var mockQueue: MockOperationQueue!
  var sut: SomeObject!
  
  override func setUp() {
    super.setUp()
    mockQueue = MockOperationQueue()
    sut = SomeObject(operationQueue: mockQueue)
  }
    
  override func tearDown() {
    mockQueue = nil
    sut = nil
    super.tearDown()
  }
  
  func test_doSomeWork_cancelsCurrentOperations() {
    sut.doSomeWork()
    XCTAssertEqual(mockQueue.cancelAllOperationsCallCount, 1)
  }
  
}
```

Knowing the exact implementation of what is being tested has the added value that a test suite can be more thorough. It's likely that most if not all code paths and edge cases are identified and tested. As mentioned in a previous post about [automating code reviews](http://www.marisibrothers.com/2016/12/automating-code-reviews.html), code coverage can help flag paths that have not been tested.

On the other hand, some of these tests will require more maintenance if they are very tied to a specific implementation. A large test suite with these kind of tests can be painful to maintain if the implementation changes frequently.

## Grey Box Testing

Sometimes it's not easy to say whether something is being tested as a black or white box, as both the interface and the implementation of a system are known. Thus, grey box testing combines the two techniques. Its objective is to assert that the interface works as documented but also test known paths within an implementation to cover the failure cases.

For example, a very common example in iOS is to convert a JSON object into a Swift `struct`:

```swift
import JSONUtilities

struct Person {

  let name: String
  let age: Int?

  init(jsonDictionary: JSONDictionary) throws {
    name = try jsonDictionary.json(atKeyPath: "nameKey")
    age = jsonDictionary.json(atKeyPath: "ageKey")
  }

}
```

This simple and very common code snippet showing how a JSON object represented in Swift is parsed can be tested in a variety of ways. The first thing to do would be to attempt to initialize this object with a known valid JSON object.

person.json:

```json
{
	"nameKey": "John Doe",
	"ageKey": 37
}
```

```swift
let validJSONDictionary = try JSONDictionary.from(filename: "person")
let person = try Person(jsonDictionary: validJSONDictionary)
XCTAssertNotNil(person)
```

This kind of test would be considered black box as we are testing that a known valid JSON object successfully creates a `Person` struct. However, for this case it's certainly valuable to check that a `Person` object does get created when the `ageKey` is missing as defined by the optional `age` property. Effectively, at this point we are white box testing since our tests would take into consideration the implementation of the `Person` initializer.

_person_without_age.json_:

```json
{
	"nameKey": "John Doe"
}
```

```swift
let validJSONDictionary = try JSONDictionary.from(filename: "person_without_age")
let person = try Person(jsonDictionary: validJSONDictionary)
XCTAssertNotNil(person)
```

# Conclusion

Ultimately, there's really no silver bullet when choosing how to test something. Before writing a test, it's important to understand whether you should test the interface, the implementation or a combination of both, and why it makes sense to do so.

*I’d like to thank [Daniela Bulgaru](http://twitter.com/danielablg), [Nahuel Marisi](http://twitter.com/nmarisi) and [Neil Horton](http://twitter.com/Neil3079) for reviewing this article.*