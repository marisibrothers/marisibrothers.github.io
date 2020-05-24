---
layout: post
title: 'Simulating signals in Swift, part 2: Mocking the accelerometer on the iOS
  Simulator'
date: '2016-02-28T22:34:00.000Z'
author: luciano_marisi
tags:
- swift
permalink: /2016/02/simulating-signals-in-swift-part-2.html
---

**TL;DR - Mock the accelerometer in the iOS simulator using "Runner"**

Following from [part 1](http://www.marisibrothers.com/2016/02/simulating-signals-in-swift-part-1.html) where I described how I implemented [Runner](https://github.com/lucianomarisi/Runner) I'll give some examples of how it can be used to mock the accelerometer in the iOS simulator. These examples can be found in the example app from the [Github](https://github.com/lucianomarisi/Runner) repository.

## CMDeviceMotion setup for mocking

A `Motionable` protocol needs to be declared so that the points being fired have `userAcceleration`, `gravity` and `rotationRate`.

```swift
protocol Motionable {
  var rotationRate: CMRotationRate { get }
  var gravity: CMAcceleration { get }
  var userAcceleration: CMAcceleration { get }
}

extension Motionable {
  var rotationRate: CMRotationRate { return CMRotationRate() }
  var gravity: CMAcceleration { return CMAcceleration() }
  var userAcceleration: CMAcceleration { return CMAcceleration() }
}
```

The `Motionable` protocol is used to replace the `CMDeviceMotionHandler` from the `CMMotionManager` class:

```swift
public typealias CMDeviceMotionHandler = (CMDeviceMotion?, NSError?) -> Void
```

This way a `Motionable` type instead of `CMDeviceMotion` object is returned:

```swift
typealias MotionableHandler = (Motionable, NSError?) -> Void
```

This means that `CMDeviceMotion` needs to conform to this protocol for the new completion handler to work:

```swift
extension CMDeviceMotion : Motionable {}
```

The first step is to create a class that will be used to interact with the `CMMotionManager`. This keeps the `CoreMotion` framework's boilerplate code on one class that can be reused in multiple places of an app.

```swift
private let defaultDeviceMotionUpdateInterval = 0.01

class DeviceMotionWrapper {
  
  private var motionManager : CMMotionManager = {
    let motionManager = CMMotionManager()
    motionManager.deviceMotionUpdateInterval = defaultDeviceMotionUpdateInterval
    return motionManager
  }()
  
  private var queue : NSOperationQueue = {
    let queue = NSOperationQueue()
    queue.maxConcurrentOperationCount = 1
    return queue
  }()

}
```

Up until this point the accelerometer would only log changes when run from a physical device since the iOS simulator does not have a built in accelerometer. Hence, the `DeviceMotionWrapper` class needs to be extended to use `Runner` to execute accelerometer points in case the iOS simulator is being used. 

A property storing an instance of `Runner` needs to be added to the `DeviceMotionWrapper` class:

```swift
private let runner = Runner()
```

...and a function to start the real accelerometer using the new `MotionableHandler` completion:

```swift
func startRealDeviceMotionUpdates(handler: MotionableHandler) {
  motionManager.startDeviceMotionUpdatesToQueue(queue) {(deviceMotion, error) -> Void in
    
    guard let deviceMotion = deviceMotion else {
      return
    }
    handler(deviceMotion, error)
  }
}
```

Finally, I defined a `struct` for the `MotionablePoint` I was going to mock that conformed to `Motionable` and `Runnable`. For this example I only mocked the `userAcceleration` but `gravity` and `rotationRate` can also be added.

```swift
struct MotionablePoint : Runnable, Motionable {
  let timestamp : NSTimeInterval
  let userAcceleration : CMAcceleration
}
```

### Case 1: Using an array of predefined points to execute

To be able to fire a predefined set of points in case the accelerometer is not there, a new generic functions needs to be added to the `DeviceMotionWrapper` class. It takes an array of both `Motionable` and `Runnable` points and either it executes these points on the iOS simulator or ignores them and defaults to the accelerometer. The `#ifdef` is used to only mock the accelerometer when the iOS simulator is running. An alternative would be to check whether the `deviceMotionAvailable` is available from the `CMMotionManager` instance.

```swift
func startDeviceMotionUpdates<PointType : protocol<Runnable, Motionable>>(mockPoints: [PointType]? = nil, timeInterval: Double = defaultDeviceMotionUpdateInterval, handler: MotionableHandler) {
  
  #if (arch(i386) || arch(x86_64)) && os(iOS)
    if let mockPoints = mockPoints {
      startMockedDeviceMotionUpdates(mockPoints, handler: handler)
      return
    }
  #endif
  
  motionManager.deviceMotionUpdateInterval = timeInterval
  startRealDeviceMotionUpdates(handler)
  
}

private func startMockedDeviceMotionUpdates<PointType : protocol<Runnable, Motionable>>(mockPoints: [PointType], handler: MotionableHandler) {
  
  runner.startWithMockPoints(mockPoints) { (mockPoint) -> Void in
    handler(mockPoint, nil)
  }
  
}
```

This would be used like this:

```swift
var mockPoints = [MotionablePoint]()

for index in 0...100 {
  let value = Double(index)
  let userAcceleration = CMAcceleration(x: value, y: value, z: value)
  let mockPoint = MotionablePoint(timestamp: NSTimeInterval(index) / 10, userAcceleration: userAcceleration)
  mockPoints.append(mockPoint)
}

deviceMotionWrapper.startDeviceMotionUpdates(mockPoints) { (point, error) -> Void in
  NSLog("\(point)")
}
```

When running this on the simulator the points will be the ones from the for loop and when running on the device the points will be the once from the device accelerometer.

### Case 2: Using a function to mock the acceleration

Similarly from the previous example a new function on `DeviceMotionWrapper` is declared to mock the accelerometer using a signal function. In this case the first argument is a function that uses a generic type for the point to run since this point needs to conform to `Motionable` and `Runnable`.

```swift
func startDeviceMotionUpdates<PointType : protocol<Runnable, Motionable>>(signalFunction: (NSTimeInterval -> PointType)? = nil, timeInterval: Double = defaultDeviceMotionUpdateInterval, handler: MotionableHandler) {
  
  #if (arch(i386) || arch(x86_64)) && os(iOS)
    if let signalFunction = signalFunction {
      startMockedDeviceMotionUpdates(signalFunction, timeInterval: timeInterval, handler: handler)
      return
    }
  #endif
  
  motionManager.deviceMotionUpdateInterval = timeInterval
  startRealDeviceMotionUpdates(handler)
  
}

private func startMockedDeviceMotionUpdates<PointType : protocol<Runnable, Motionable>>(signalFunction: (NSTimeInterval -> PointType),  timeInterval: Double = defaultDeviceMotionUpdateInterval, handler: MotionableHandler) {
  
  runner.startWithFunction(signalFunction, timeInterval: timeInterval) { (mockPoint) -> Void in
    handler(mockPoint, nil)
  }
  
}
```

Similarly as in the previous example, this would be used like this:

```swift
deviceMotionWrapper.startDeviceMotionUpdates(sineSignal, timeInterval: 0.1) { (point, error) -> Void in
  NSLog("\(point.userAcceleration)")
}
```

The sine signal function as explained in the previous [post](http://www.marisibrothers.com/2016/02/simulating-signals-in-swift-part-1.html) is defined as:

```swift
// A.sin(f.t+phaseShift)+offset
func sineSignal(nextTimestamp: NSTimeInterval) -> MotionablePoint {
  let signalFrequency = 1.0
  let amplitude = 2.0
  let offset = 0.5
  let phaseShift = 0.2
  let value = amplitude * sin(nextTimestamp * signalFrequency + phaseShift) + offset
  let userAcceleration = CMAcceleration(x: value, y: value, z: value)
  return MotionablePoint(timestamp: nextTimestamp, userAcceleration: userAcceleration)
}
```

It may be clearer to download and have a look at all the code together to understand the whole implementation. This can be found on [Github](https://github.com/lucianomarisi/Runner).