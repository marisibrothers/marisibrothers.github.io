---
layout: post
title: Protocol oriented loading of resources from a network service in Swift
date: '2016-07-10T16:28:00.002+01:00'
author: luciano_marisi
tags:
- software architecture
- iOS
- swift
- design patterns
permalink: /2016/07/protocol-oriented-loading-of-resources.html
---

**Update 12-December-2016: [TABResourceLoader](https://github.com/theappbusiness/TABResourceLoader) was created using this pattern and is production ready.**

Using protocol oriented programming to load resources from the network
**TL;DR - Define the resources in your application by conforming to protocols that define where and how to get them using a generic service type. Finally make your operations conform to a protocol to use them to retrieve your resources**

With the addition of protocol extensions in Swift 2.0 it's now possible to focus less on subclassing and define your interfaces with default implementations on a protocol extension. This concept is used extensibly in the approach I currently use to a load resources into an iOS app. The sample code can be downloaded from [Github](https://github.com/lucianomarisi/LoadIt).

## Defining a Resource

A resource can be defined as something that you application will consume.
For example, it could be a JSON or XML file, an image, a video, etc. In Swift this can be represented using a protocol with an associated type:

```swift
public protocol ResourceType {
  associatedtype Model
}
```

The `Model` would be the strong type used to represent that data in your code. Specifically, we can then define a JSON resource. It conforms `ResourceType` and contains two parsing functions, a way of retrieving the generic `Model` from a JSON dictionary and a way of parsing it from a JSON array.

```swift
public protocol JSONResourceType: ResourceType {
  func modelFrom(jsonDictionary jsonDictionary: [String : AnyObject]) -> Model?
  func modelFrom(jsonArray jsonArray: [AnyObject]) -> Model?
}
```

Naturally for most cases a resource will come from either a dictionary or an array so I use a protocol extension to add default implementations of this functions to return nil. When you create a new type that conforms to `JSONResourceType` you should provide an implementation to either of these functions.

```swift
extension JSONResourceType {
  public func modelFrom(jsonDictionary jsonDictionary: [String : AnyObject]) -> Model? { return nil }
  public func modelFrom(jsonArray jsonArray: [AnyObject]) -> Model? { return nil }
}
```

The following step is to transform some `NSData` into either a JSON dictionary (i.e. `[String: AnyObject]`) or a JSON array (i.e. `[AnyObject]`). This is done using the commonly known `NSJSONSerialization.JSONObjectWithData`. As a side note, I prefer to use a `Result` type as the return type since this works very well for operations where the return type is different depending whether the function succeeds or fails:

```swift
public enum Result<T> {
  case Success(T)
  case Failure(ErrorType)
}
```

```swift
enum ParsingError: ErrorType {
  case InvalidJSONData
  case CannotParseJSONDictionary
  case CannotParseJSONArray
  case UnsupportedType
}
```

```swift
extension JSONResourceType {
  
  func resultFrom(data data: NSData) -> Result<Model> {
    guard let jsonObject = try? NSJSONSerialization.JSONObjectWithData(data, options: .MutableContainers) else {
      return .Failure(ParsingError.InvalidJSONData)
    }
    
    if let jsonDictionary = jsonObject as? [String: AnyObject] {
      return resultFrom(jsonDictionary: jsonDictionary)
    }
    
    if let jsonArray = jsonObject as? [AnyObject] {
      return resultFrom(jsonArray: jsonArray)
    }
    
    // This is likely an impossible case since `JSONObjectWithData` likely only returns [String: AnyObject] or [AnyObject] but still needed to appease the compiler
    return .Failure(ParsingError.UnsupportedType)
  }
  
  private func resultFrom(jsonDictionary jsonDictionary: [String: AnyObject]) -> Result<Model> {
    if let parsedResults = modelFrom(jsonDictionary: jsonDictionary) {
      return .Success(parsedResults)
    } else {
      return .Failure(ParsingError.CannotParseJSONDictionary)
    }
  }
  
  private func resultFrom(jsonArray jsonArray: [AnyObject]) -> Result<Model> {
    if let parsedResults = modelFrom(jsonArray: jsonArray) {
      return .Success(parsedResults)
    } else {
      return .Failure(ParsingError.CannotParseJSONArray)
    }
  }
  
}
```

At this point we can load any `JSONResourceType` from some `NSData`, so the next step is to get the data from somewhere. For the sake of keeping an already long post short, I'm only going to show how to retrieve it from a web service.

Before diving into the networking logic I'll introduce another protocol to define the necessary values to retrieve a `JSONResourceType` from the network. In most scenarios the following 4 parameters will be sufficient to download the resource.

```swift
public protocol NetworkResourceType {
  var url: NSURL { get }
  var HTTPMethod: String { get }
  var allHTTPHeaderFields: [String: String]? { get }
  var JSONBody: AnyObject? { get }
}
```

In a lot of cases only the URL is necessary to know where the JSON file is so we can add default values for the other three values:

```swift
public extension NetworkResourceType {
  var HTTPMethod: String { return "GET" }
  var allHTTPHeaderFields: [String: String]? { return nil }
  var JSONBody: AnyObject? { return nil }
}
```

The final step is to add the ability to generate a `NSURLRequest` from these parameters:

```swift
extension NetworkResourceType {
  func urlRequest() -> NSURLRequest {
    let request = NSMutableURLRequest(URL: url)
    request.allHTTPHeaderFields = allHTTPHeaderFields
    request.HTTPMethod = HTTPMethod
    
    if let body = JSONBody {
      request.HTTPBody = try? NSJSONSerialization.dataWithJSONObject(body, options: NSJSONWritingOptions.PrettyPrinted)
    }
    
    return request
  }
}
```

At this point we can define a `NetworkJSONResource` by combining `NetworkResource` and `JSONResource`:

```swift
public protocol NetworkJSONResourceType: NetworkResourceType, JSONResourceType {}
```

By conforming to `NetworkJSONResourceType` we know where the location of the JSON response is and how to map it into an app specific data type.

## Downloading the resource from the network

Now that we know how a `NetworkJSONResourceType` looks it's time to download it! This is done using the `NSURLSession.sharedSession`. The first thing to do is to declare a protocol that defines how a resource service looks like. It has 2 functions, one for initialization and another for fetching the resource with a completion handler:

```swift
public protocol ResourceServiceType {
  associatedtype Resource: ResourceType
  init()
  func fetch(resource resource: Resource, completion: (Result<Resource.Model>) -> Void)
}
```

Then we can create a `NetworkJSONService` that conforms to this `ResourceServiceType` to download the resource. 

*As an aside, before diving into the `NetworkJSONService` I define a `URLSessionType ` protocol that wraps `NSURLSession`'s `dataTaskWithRequest` method, so that I can then use dependency injection to test the service by passing in a `URLSessionType ` type.*

```swift
protocol URLSessionType {
  func performRequest(request: NSURLRequest, completion: (NSData?, NSURLResponse?, NSError?) -> Void)
}

extension NSURLSession: URLSessionType {
  public func performRequest(request: NSURLRequest, completion: (NSData?, NSURLResponse?, NSError?) -> Void) {
    dataTaskWithRequest(request, completionHandler: completion).resume()
  }
}
```

The implementation of the `NetworkJSONService` service is very simple.
In the `fetch(resource:completion:)` method from `ResourceServiceType` the `URLSessionType` performs a request and calls the completion handler with the `Result`. The handling of the response is done in the private  `resultFrom(resource data:URLResponse:error:)` method. At this point the usual checking for errors is carried out (error checking could be more exhaustive). Finally the `JSONResourceType` method defined previously to convert the `NSData` onto the needed model is invoked.

```swift
enum NetworkJSONServiceError: ErrorType {
  case NetworkingError(error: NSError)
  case NoData
}

public struct NetworkJSONService<Resource: NetworkJSONResourceType> {
  
  private let session: URLSessionType
  
  init(session: URLSessionType) {
    self.session = session
  }
  
  private func resultFrom(resource resource: Resource, data: NSData?, URLResponse: NSURLResponse?, error: NSError?) -> Result<Resource.Model> {
    if let error = error {
      return .Failure(NetworkJSONServiceError.NetworkingError(error: error))
    }
    
    guard let data = data else {
      return .Failure(NetworkJSONServiceError.NoData)
    }
    
    return resource.resultFrom(data: data)
  }
  
}
```

```swift
// MARK: - ResourceServiceType
extension NetworkJSONService: ResourceServiceType {
  
  public init() {
    self.init(session: NSURLSession.sharedSession())
  }
  
  public func fetch(resource resource: Resource, completion: (Result<Resource.Model>) -> Void) {
    let urlRequest = resource.urlRequest()
    
    session.performRequest(urlRequest) { (data, _, error) in
      completion(self.resultFrom(resource: resource, data: data, URLResponse: nil, error: error))
    }
  }
  
}
```

That's pretty much all that is needed to have a very flexible and testable networking stack where all the endpoints called are defined as resources.

## Using an operation to retrieve the data from the network

If you use `NSOperation`s, this pattern fits very well with them. If you don't know much about them I would highly encourage watching the [Advanced Operations WWDC 2016 video](https://developer.apple.com/videos/play/wwdc2015/226/) and checking out the [sample code](https://developer.apple.com/sample-code/wwdc/2015/downloads/Advanced-NSOperations.zip).

From this point I'll assume that you have a basic understanding of how to use operations. The first thing to do is define two protocols, `Cancellable` to know whether the operation has been cancelled and `Finishable` to be called when the work in the operation has been completed. An `NSOperation` will conform to the first protocol, so no extra work is needed to conform to it, the second one is inspired by the `Operation` class in the *Advanced Operations* sample code where the subclass of the operation must call `finish` with some errors if any exist when the work is complete.

```swift
public protocol Cancellable: class {
  var cancelled: Bool { get }
}

public protocol Finishable: class {
  func finish(errors: [NSError])
}
```

The next protocol defines what a `ResourceOperationType` should do. This conforms to both `Cancellable` and `Finishable`, and it include an associated type that conforms to the `ResourceType` protocol defined previously. The other requirements are a method to fetch the associatedtype `Resource` using a `ResourceServiceType` and a method that will be called when the operation completes providing the `Result` of it.

```swift
public protocol ResourceOperationType: Cancellable, Finishable {
  associatedtype Resource: ResourceType
  func fetch<Service: ResourceServiceType where Service.Resource == Resource>(resource resource: Resource, usingService service: Service)
  func didFinishFetchingResource(result result: Result<Resource.Model>)
}
```

Noticeably, the `ResourceOperationType` protocol does not provide a definition of where the `Resource` needs to be retrieved from, it could come form somewhere in the disk, the cloud, etc. Only it defines that it can be retrieved using a `ResourceServiceType`. To avoid having to write the code that fetches the resource using a service on each operation a function can be implemented on an extension on `ResourceOperationType` to be called in the method where the operation does it's work. (For `NSOperation`s this can be called on the `main()` function).

```swift
public extension ResourceOperationType {
  
  public func fetch<Service: ResourceServiceType where Service.Resource == Resource>(resource resource:Resource, usingService service: Service) {
    if cancelled { return }
    service.fetch(resource: resource) { [weak self] (result) in
      guard let strongSelf = self else { return }
      if strongSelf.cancelled { return }
      NSThread.executeOnMain { [weak self] in
        guard let strongSelf = self else { return }
        if strongSelf.cancelled { return }
        strongSelf.finish([])
        strongSelf.didFinishFetchingResource(result: result)
      }
    }
  }
  
}
```

That's it! A few protocol definitions, some enums and one struct later we can create our operations that conform to `ResourceOperationType` and use a `NetworkJSONService` to download our JSON in a concurrent way.

## A concrete example

After writing a lot of code and explaining the various patterns of this architecture it's time to share a concrete example to see how this works in the real world. Let's say we are downloading a list of cities from some web service where the JSON response looks like this:

```json
{
  "cities": [{
    "name": "Paris"
  }, {
    "name": "London"
  }]
}
```

The model in our application can be represented by the following struct:

```swift
struct City {
  let name: String
}
```

The parsing can be done in an extension where the `City` can be initialised from a JSON dictionary. (Note there are simpler ways to write the parsing logic, check out [JSONUtilities](https://github.com/lucianomarisi/JSONUtilities) for an example)

```swift
extension City {
  init?(jsonDictionary: [String : AnyObject]) {
    guard let parsedName = jsonDictionary["name"] as? String else {
      return nil
    }
    name = parsedName
  }
}
```

After creating the model we need to create the `CitiesResource` where we define how to decode the 'cities` key into an array of JSON dictionary cities.

```swift
// This base endpoint would probably be defined somewhere globally
let baseURL = NSURL(string: "http://localhost:8000/")!

struct CitiesResource: NetworkJSONResourceType {
  typealias Model = [City]
  
  let url: NSURL
    
  init(continent: String) {
    url = baseURL.URLByAppendingPathComponent("\(continent).json")
    filename = continent
  }
  
  //MARK: JSONResource
  func modelFrom(jsonDictionary jsonDictionary: [String: AnyObject]) -> [City]? {
    guard let
      citiesJSONArray: [[String: AnyObject]] = jsonDictionary["cities"] as? [[String: AnyObject]]
      else {
        return []
    }
    return citiesJSONArray.flatMap(City.init)
  }
  
}
```

Once all the information is defined about how to parse the `cities` endpoint into an array of cities (i.e. `[City]`) it's time to download them. To do this I have `BaseOperation` which is a subclass of `NSOperation` where I have all the finished, executing and canceled KVO boilerplate (see the implementation on [Github](https://github.com/lucianomarisi/LoadIt)). Nothing fancy, the only thing needed to know is that the work is carried out on the `execute()` function. Again I highly encourage to watch the Advanced Operations WWDC video as this `BaseOperation` can be replaced by the `Operation` Apple provides in the sample code. 

The operation is initialised with a continent (maybe something that the user selected from a table) to be able to create the `CitiesResource`. To notify about the completion of the operation I use a completion handler containing the generic operation and result. 

```swift
final class ResourceOperation<ResourceService: ResourceServiceType>: BaseOperation, ResourceOperationType {
  
  typealias Resource = ResourceService.Resource
  typealias DidFinishFetchingResourceCallback = (ResourceOperation<ResourceService>, Result<Resource.Model>) -> Void

  private let resource: Resource
  private let service: ResourceService  
  private let didFinishFetchingResourceCallback: DidFinishFetchingResourceCallback
  
  init(resource: ResourceService.Resource, service: ResourceService = ResourceService(), didFinishFetchingResourceCallback: DidFinishFetchingResourceCallback) {
    self.resource = resource
    self.service = service
    self.didFinishFetchingResourceCallback = didFinishFetchingResourceCallback
    super.init()
  }
  
  override func execute() {
    fetch(resource: resource, usingService: service)
  }
  
  func didFinishFetchingResource(result result: Result<Resource.Model>) {
    didFinishFetchingResourceCallback(self, result)
  }
  
}
```

The definition of the operation for the `CitiesResource` that uses a `NetworkJSONService` can be declared in a `typealias`

```swift
typealias CitiesNetworkResourceOperation = ResourceOperation<NetworkJSONService<CitiesResource>>
```

At this point the networking stack is complete and can be used wherever it makes sense in your application. For the sake of simplicity we'll put in a view controller following the commonly used MVC pattern (i.e. Massive View Controller):

```swift
class ViewController: UIViewController {

  private let operationQueue = NSOperationQueue()

  override func viewDidLoad() {
    super.viewDidLoad()    
    let americaResource = CitiesResource(continent: "america")
    let citiesNetworkResourceOperation = CitiesNetworkResourceOperation(resource: americaResource) { [weak self] operation, result in
      if operation.cancelled { return }
      // Handle result if operation not cancelled
    }
    operationQueue.addOperation(citiesNetworkResourceOperation)
  }
     
}
```

# Conclusions

There is very little repetition in the components built with this architecture. It's very easy to test, everything in this post apart from the `ViewController` can be easily unit tested (mainly through dependency injection). The components are small and easy to reason about, meaning onboarding of new developers is simpler. The ownership of each object is simple, so once the operation is torn down, the whole stack is removed from memory. Finally, adding this pattern to a current architecture is straight forward because each resource has it's own networking stack independent of the others.

To see how this pattern can be use to load a JSON file from disk see the [Github](https://github.com/lucianomarisi/LoadIt) repository where the code for `LoadIt` can be found.