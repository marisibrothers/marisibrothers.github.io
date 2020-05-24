---
layout: post
title: Common unit testing techniques on iOS
date: '2017-03-13T19:40:00.000Z'
author: luciano_marisi
tags:
- iOS
- swift
- unit testing
permalink: /2017/03/common-unit-testing-techniques-on-ios.html
---

**TL;DR - Most if not all unit test cases on iOS can follow the same commonly known pattern: GIVEN a set of initial conditions, WHEN something happens, THEN something is expected.**

*Note: Code snippets are written in Swift 3 using `XCTest` assertions*

## Preamble: Key definitions

The structure of a unit test almost always follows this [pattern](https://martinfowler.com/bliki/GivenWhenThen.html):

1. **Given** a set of initial conditions
- **When** something happens
- **Then** something is expected

The object being tested is generally referred as the [system under test (SUT)](https://en.wikipedia.org/wiki/System_under_test). Objects that interact with the SUT and are needed to be able to write a unit test are called [test doubles](https://en.wikipedia.org/wiki/Test_double). I'll use the term *mock* instead of *test double* since it's more commonly used even though [technically](https://martinfowler.com/articles/mocksArentStubs.html) it's not correct. Swift does not have mocking frameworks because reflection is limited and for pure swift objects it's not possible to change the implementation of methods. Therefore, in Swift we need to implement our mocks manually. A real object can be substituted by a mock using [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection).

## Typical kind of unit tests

Unit testing in iOS has come a long way since the iOS SDK was released in 2008. It's no longer rare to see iOS developers unit testing the majority of the code they write. The following are some of the most common cases for unit tests:

1. Assert a method returns an expected value given:
	- [An input](/2017/03/common-unit-testing-techniques-on-ios.html#1a)
	- [The state of a dependency](/2017/03/common-unit-testing-techniques-on-ios.html#1b)
2. [Assert properties instantiated depending on parameters](/2017/03/common-unit-testing-techniques-on-ios.html#2)
3. [Assert a method in a mock gets called](/2017/03/common-unit-testing-techniques-on-ios.html#3)
4. Assert that calling a method in the SUT has a side effect in:
	- [The SUT](/2017/03/common-unit-testing-techniques-on-ios.html#4a)
	- [A mock](/2017/03/common-unit-testing-techniques-on-ios.html#4b)
5. Assert that a change in a (mocked) dependency has a side effect in the SUT


## Unit Test Examples

<a name="1a"></a>
### 1a. Assert a method returns an expected value given an input:

A method that returns the element in a array if it exists could look something like this:

```swift
extension Array {
	subscript (safe index: Int) -> Element? {
		return indices ~= index ? self[index] : nil
	}
}
```

The unit tests for this method can be done by treating the SUT as a [black box](/2017/01/the-yin-and-yang-of-unit-testing-in-ios.html). The following test class shows how three test cases can be written to test the behaviour of the safe index function. The code is intentionally verbose to demonstrate the example better.

```swift
class ArrayTests: XCTestCase {

	func testIndexWithinBoundsReturnsElement() {
		// GIVEN
		let sut = [1, 2, 3]
		
		// WHEN
		let itemAtIndex = sut[safe: 1]
		
		// THEN
		XCTAssertEqual(itemAtIndex, 2)
	}
	
	func testNegativeIndexReturnsNil() {
		let sut = [1, 2, 3]
		let itemAtIndex = sut[safe: -1]
		XCTAssertNil(itemAtIndex)
	}
	
	func testOutOfboundsIndexReturnsNil() {
		let sut = [1, 2, 3]
		let itemAtIndex = sut[safe: 3]
		XCTAssertNil(itemAtIndex)
	}
}
```

<a name="1b"></a>
### 1b. Assert a method returns an expected value given the state of a dependency

The business logic for deciding whether to show an alert to the user requesting for location permissions could be implemented like this:

```swift
struct OnboardingState_Untestable {
	func shouldShowEnableLocationAlert() -> Bool {
		guard CLLocationManager.locationServicesEnabled() else { return true }
		return false
	}
}
```

The problem of this implementation is that the `shouldShowEnableLocationAlert` method uses `CLLocationManager` internally to return it's output. Therefore, to test this method we need mock and inject this dependency. Defining a protocol that `CLLocationManager` can automatically conform to:

```swift
protocol LocationManagerType: class {
	static func locationServicesEnabled() -> Bool
}
extension CLLocationManager: LocationManagerType {}
```

Modifying `shouldShowEnableLocationAlert` to pass the `LocationManagerType` type as a parameter defaulting to the `CLLocationManager` will allow to mock the dependency in a unit test. Note that this does not affect how this method is used in production. This kind of dependency injection is called **[Interface injection](https://en.wikipedia.org/wiki/Dependency_injection#Interface_injection)** and in this case we are injecting a type as opposed to an instance because the `CLLocationManager` method we need to mock is a class function. The updated `OnboardingState` would be:

```swift
struct OnboardingState {
	func shouldShowEnableLocationAlert(locationManager: LocationManagerType.Type = CLLocationManager.self) -> Bool {
		guard locationManager.locationServicesEnabled() else { return true }
		return false
	}
}
```

Mocking a class function as opposed to an instance function is generally messier because classes *are* singletons. There is only one class definition per instance of a program. Hence, the mocked state needs to be global. Using protocol conformance we can mock the `CLLocationManager`:

```swift
class MockedLocationManager: LocationManagerType {
	static var mockedLocationServicesEnabled = true
	static func locationServicesEnabled() -> Bool {
		return mockedLocationServicesEnabled
	}
}
```

A subclass of `XCTestCase` can override the `setUp` and `tearDown` methods. These functions get called before and after each test. It's common to initialise the SUT on `setUp` but not necessary. It's good practice to implement the `tearDown` method to deinitialise your SUT and mocks, specially the reference type ones. Otherwise, these objects will continue to exists while other tests run potentially interfering with them. This [post](http://qualitycoding.org/teardown/) explains possible issues in more detail. The 2 test cases for the `OnboardingState` can be:

```swift
class OnboardingStateTests: XCTestCase {
	
	var sut: OnboardingState!
	
	override func setUp() {
		super.setUp()
		// GIVEN
		sut = OnboardingState()
	}
	
	override func tearDown() {
		sut = nil
		super.tearDown()
	}
	
	func test_shouldShowEnableLocationAlert_returnsTrue_when_locationServicesAreDisabled() {
		// WHEN
		MockedLocationManager.mockedLocationServicesEnabled = false

		// THEN
		XCTAssert(sut.shouldShowEnableLocationAlert(locationManager: MockedLocationManager.self))
	}
	
	func test_shouldShowEnableLocationAlert_returnsFalse_when_locationServicesAreEnabled() {
		MockedLocationManager.mockedLocationServicesEnabled = true
		XCTAssertFalse(sut.shouldShowEnableLocationAlert(locationManager: MockedLocationManager.self))
	}
}
```

<a name="2"></a>
### 2. Assert properties instantiated depending on parameters

Let's say that a person is represent by the following `struct` and the JSON parsing is implemented in a similar way as [Apple](https://developer.apple.com/swift/blog/?id=37) explains.

```swift
struct Person {
	let name: String
	let age: Int
}

extension Person {
	init?(dictionary: [String: Any]) {
		guard let name = dictionary["name"] as? String,
			  let age = dictionary["age"] as? Int else {
				return nil
		}
		self.name = name
		self.age = age
	}
}
```

One of the tests would be to assert that for a given valid dictionary a valid `Person` is created. Personally, I don't mind having more than one assertion in a test if they are related. For example, the following test asserts both the name and age properties:

```swift
class PersonTests: XCTestCase {

	func test_allPropertiesAreSetCorrectlyForAValidDictionary() {
		// GIVEN
		let validDictionary: [String: Any] = ["name": "John Doe", "age": 35 ]
		
		// WHEN
		let sut = Person(dictionary: validDictionary)
		
		// THEN
		XCTAssertEqual(sut?.name, "John Doe")
		XCTAssertEqual(sut?.age, 35)
	}
}
```

<a name="3"></a>
### 3. Assert a method in a mock gets called

Commonly we need to download images from a URL, for example when displaying them in a `UITableView`. This task can be done by the following `ImageFetcher` which uses an `OperationQueue` and adds some kind of image operation that takes a URL and performs a network request to retrieve it. If we pop this screen we probably want to cancel all current image operations as they are no longer relevant. To do this we call the `cancelAllOperations` method on the queue.

```swift
class ImageFetcher {
	
	private let operationQueue: OperationQueue
	init(operationQueue: OperationQueue = OperationQueue()) {
		self.operationQueue = operationQueue
	}
	
	func fetch(imageURL: URL, completion: (UIImage?) -> Void) {
		// some implementation adding an image operation to the queue
	}
	
	func cancelFetchingAllImages() {
		operationQueue.cancelAllOperations()
	}
}
```

To be able to test this we pass the `OperationQueue` in the initialiser, this is know as **[Constructor injection](https://en.wikipedia.org/wiki/Dependency_injection#Constructor_injection)**. Using inheritance we can mock the `OperationQueue` and count the number of times `cancelAllOperations` is called. In our test case we can assert that `cancelAllOperations` gets called exactly once when `cancelFetchingAllImages` is called.

```swift
class MockOperationQueue: OperationQueue {
	var cancelAllOperationsCountCallCount = 0
	override func cancelAllOperations() {
	 	// In this case super is called to avoid having
	 	// side effects that are not true
		super.cancelAllOperations()
		cancelAllOperationsCountCallCount += 1
	}
}

class ImageFetcherTests: XCTestCase {
	
	var sut: ImageFetcher!
	var mockOperationQueue: MockOperationQueue!
	
	override func setUp() {
		super.setUp()
		// GIVEN
		mockOperationQueue = MockOperationQueue()
		sut = ImageFetcher(operationQueue: mockOperationQueue)
	}
	
	override func tearDown() {
		mockOperationQueue = nil
		sut = nil
		super.tearDown()
	}
	
	func test_cancelFetchingAllImages_calls_cancelAllOperations() {
		// WHEN
		sut.cancelFetchingAllImages()

		// THEN
		XCTAssertEqual(mockOperationQueue.cancelAllOperationsCountCallCount, 1)
	}
}
```

<a name="4a"></a>
### 4a. Assert that calling a method in the SUT has a side effect in the SUT

*Note: The code in this example is for illustrative purposes as some implementations are missing.*

A very common pattern in iOS is to have a model object such as the previously defined `Person` and use it setup a custom view. For example:

```swift
struct Model { /* some properties */ }

class View: UIView {
	func configure(with model: Model) { /* configure the view */ }
}
```

This kind of example is best tested using screenshot testing instead of asserting each and every property that would be changed in the `View` by the `Model`. The test is simpler to write and if the `View`'s implementation changed the differences could be seen clearly by inspecting the before and after screenshots of the view. This can be done using [FBSnapshotTestCase](https://github.com/facebook/ios-snapshot-test-case), for example one test case for a `Model` with predefined properties:

```swift
class ViewTests: FBSnapshotTestCase {
	
	var sut: View!
	
	override func setUp() {
		super.setUp()
		sut = View()
	}
	
	override func tearDown() {
		sut = nil
		super.tearDown()
	}
	
	func test_ViewWithModel() {
		// GIVEN
		let model = Model( /* initialisation with mocked parameter*/ )
		
		// WHEN
		sut.configure(with: model)
		
		// THEN
		FBSnapshotVerifyView(sut)
	}
}
```

<a name="4b"></a>
### 4b. Assert that calling a method in the SUT has a side effect in a mock.

Assuming we have a view controller with a table view displaying a list of strings. When a cell is tapped the view controller notifies it's delegate about this and passes the item selected. The code looks something like this:

```swift
protocol SelectionViewControllerDelegate: class {
	func didSelect(_ selectionViewController: SelectionViewController, item: String)
}

class SelectionViewController: UIViewController, UITableViewDelegate {
	
	private weak var delegate: SelectionViewControllerDelegate?
	private let items: [String]
	
	init(items: [String], delegate: SelectionViewControllerDelegate) {
		self.items = items
		self.delegate = delegate
		super.init(nibName: nil, bundle: nil)
	}
	
	func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
		let item = items[indexPath.row]
		delegate?.didSelect(self, item: item)
	}
	
	required init?(coder aDecoder: NSCoder) { fatalError() }
}
```

To test the delegate pattern communication, the `SelectionViewControllerDelegate` can be mocked by creating an object that conforms to it and storing the `item`. The `MockSelectionViewControllerDelegate` can be injected using constructor injection. The `capturedItem` can then be used in a test case to assert that selecting a row in the tableView calls the delegate method with the correct item:

```swift
class MockSelectionViewControllerDelegate: SelectionViewControllerDelegate  {
	var capturedItem: String?
	func didSelect(_ selectionViewController: SelectionViewController, item: String) {
		capturedItem = item
	}
}

class SelectionViewControllerTests: XCTestCase {
	
	var sut: SelectionViewController!
	var mockDelegate: MockSelectionViewControllerDelegate!
	
	override func setUp() {
		super.setUp()
		mockDelegate =  MockSelectionViewControllerDelegate()
	}
	
	override func tearDown() {
		mockDelegate = nil
		sut = nil
		super.tearDown()
	}
	
	func test_tableViewDidSelectRowtAtIndexPath_calls_delegateWithSelectedItem() {
		// GIVEN
		let mockItems = ["a", "b", "c"]
		sut = SelectionViewController(items: mockItems, delegate: mockDelegate)

		// WHEN
		sut.tableView(UITableView(), didSelectRowAt: IndexPath(row: 1, section: 0))
		
		// THEN
		XCTAssertEqual(mockDelegate.capturedItem, "b")
	}
}
```

<a name="5"></a>
### 5. Assert that a change in a (mocked) dependency has a side effect in the SUT

An object used to fetch data from a URL that used the shared `URLSession` would look something like this:

```swift
enum Result<T> {
	case success(T)
	case failure(Error?)
}

class HTTPClient_Untestable {
	func fetchData(forURL url: URL, completion: @escaping (Result<Data>) -> Void) {
		// Use URLSession.shared to make a network request
	}
}
```

This object is not testable because it uses an internal dependency that cannot be accessed. Therefore, to extract the `URLSesssion` out we first declare a protocol that `URLSession` can conform to:

```swift
protocol URLSessionType {
	func dataTask(with request: URLRequest, completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTask
}
extension URLSession: URLSessionType {}
```

The updated `HTTPClient` now takes in a `URLSessionType` in the initialiser and the `fetchData` method uses this injected session object.

```swift
class HTTPClient {
	
	private let session: URLSessionType
	init(session: URLSessionType = URLSession.shared) {
		self.session = session
	}
	
	func fetchData(forURL url: URL, completion: @escaping (Result<Data>) -> Void) {
		let request = URLRequest(url: url)
		let task = session.dataTask(with: request) { (data, _, error) in
			guard let data = data else {
				completion(Result.failure(error))
				return
			}
			completion(Result.success(data))
		}
		task.resume()
	}
}
```

To test this, we mock the `URLSessionDataTask` because it's an abstract class so `resume` needs to be overridden otherwise an exception would be thrown. Then we create a mock that conforms to `URLSessionType`. This mock stores the completion handler sent from the SUT.

```swift
class MockURLSessionDataTask: URLSessionDataTask {
	override func resume() {}
}

class MockURLSession: URLSessionType {
	var capturedCompletion: ((Data?, URLResponse?, Error?) -> Void)?
	func dataTask(with request: URLRequest, completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTask {
		capturedCompletion = completionHandler
		return MockURLSessionDataTask()
	}
}
```

This last test case uses [`XCTestExpectation`](https://developer.apple.com/reference/xctest/xctestexpectation) because the `fetchData` function calls a completion handler asynchronously. Unfortunately, this means that the *THEN* part of the test partly needs to be defined before the *WHEN*. Note that we call the stored completion handler on the *WHEN* part to simulate a successful response from the session.

```swift
class HTTPClientTests: XCTestCase {
	
	var mockURLSession: MockURLSession!
	var sut: HTTPClient!
	
	override func setUp() {
		super.setUp()
		mockURLSession = MockURLSession()
		sut = HTTPClient(session: mockURLSession)
	}
	
	override func tearDown() {
		mockURLSession = nil
		sut = nil
		super.tearDown()
	}
	
	func test_fetchData_calls_completionWithSuccessResult_whenDataIsReturnedFromSession() {
		// GIVEN
		let mockURL = URL(string: "www.test.com")!
		let expectation = self.expectation(description: #function)
		sut.fetchData(forURL: mockURL) { result in
			// THEN (Partly defined before WHEN because of asynchronous nature of test)
			switch result {
			case .success(let data):
				XCTAssertEqual(data, Data())
			case .failure(let error):
				XCTFail("Unexpected failure with error: \(error)")
			}
			expectation.fulfill()
		}
		
		// WHEN
		mockURLSession.capturedCompletion?(Data(), nil, nil)
		
		// THEN (continued)
		waitForExpectations(timeout: 1, handler: nil)
	}
}
```

*I’d like to thank [Nahuel Marisi](http://twitter.com/nmarisi) and [Neil Horton](http://twitter.com/Neil3079) for reviewing this article.*