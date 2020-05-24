---
layout: post
title: 'Introduction to ReactiveCocoa 4: Part 1'
date: '2016-07-03T21:41:00.001+01:00'
author: nahuel_marisi
tags:
- swift
- ReactiveCocoa
- RFP
- functional programming
permalink: /2016/07/introduction-to-reactivecocoa-4-part-1.html
---

**TL;DR - ReactiveCocoa 4 is a reactive functional programming framework: Streams of events (such as button clicks or data from the network) called signals, are manipulated using functional constructs like filter, map, reduce, and so on. The whole idea is to react to events (instead of doing polling for example) and reduce state as much as possible.**

### What is the purpose of this article?

This blog post is the first in a series of articles about Functional Reactive Programming using ReactiveCocoa 4. The idea is to gradually introduce FRP and ReactiveCocoa, hopefully leading to its adoption by the readers. We won't go through the installation of ReactiveCocoa, as this is well documentated on their [website](https://github.com/ReactiveCocoa/ReactiveCocoa).

## What is FRP?

NB: If you're only interested in ReactiveCocoa, you can skip this section.

Functional reactive programming (FRP) is the combination of reactive and functional programming techniques. Functional programming in the context of **Swift** is the use of functional constructs such as `map`, `filter`, `reduce`, and so on. The idea is to avoid changing the state of the inputs to these functions, eliminating side effects and therefore reducing bugs. Functional programming is declarative, meaning that we describe the desired outcome instead of its control flow. Let's look at a very simple example, a `for` loop:

```swift
var array = [1, 2, 3]
for index in 0..<array.count {
    array[index] += 1
}
```
The `for` loop modifies the original array, so in other words it has a side effect. If some other part
of the program depended on values of this array, we could end up with some strange behaviour given that the
value of the array are being changed without notice.

A functional way, using `map` has no side effects:

```swift
let newArray = array.map { $0 + 1 }
```

`newArray` is a copy, and therefore any other part of the code that depends on `array` will suffer no side effects. This very simple example illustrates the advantages of using functional constructs. FRP is all about applying these constructs to stream of events (called signals). 


# ReactiveCocoa

The following section will describe in detail the main building blocks of FRP using the ReactiveCocoa framework.

## Events

Events are the fundamental building blocks of ReactiveCocoa. An event is emitted by an action such as a button being clicked, data from the network and so on. There are four types of events:

### Next
A `Next` event is usually emitted when something has happened (such as a button click). This type of event can carry a payload with it, which can be any object, or it can also carry no payload so the type of event would be void. For example, if a button is pressed we might only care about that fact, so there would be no payload. However, if the event is the next value of a `UITextField`, the payload might be the `String` value of the text.

### Completed
When a source will not send any more data (for example, a file download is complete) a `Completed` event can be sent. This will be the last event sent as the signal from the source will be disposed of after this (see Disposing signal further down for more details). 


### Failed
If an error occurs (such as not being able to contact the server), we can represent this with an `Error` event. The event contains an error (usually of type `NSError`) with the information of why the failure occurred. This type of even will also dispose of the signal and therefore no further events will be sent.


### Interrupted
This is not a very common type of event, but it can be used to represent an interruption (such as the user cancelling a download) in a stream of events. It carries no information. As `Failed` and `Completed` do, interrupted also disposes of the signal, meaning that no further events should be expected.

## Signals
A signal is a stream of events. Over time, a signal will emit various `Next` events, followed by a `Completed` event if there were no errors. You can think of a signal as an array of events, except that we have no list of historical values.


### Example
To illustrate the various ways of manipulating signals, I've built a demo application that you can retrieve from [github](https://github.com/nmarisi/RacTest1). 

<p align="center">
   <img src="/assets/images/RAC_p1_demo.png"/>
</p>

This example installs ReactiveCocoa via cocoapods, but I've included the `Pods/` directory so you only need to open the workspace to build the project.

As you can see from the screenshot, the example has:
- A counter label that will be updated from a signal 
- A `UITextField` whose text signal is transformed into a signal that emits events with capitalized strings.
- Another `TextField` whose signal is filtered for mobile phone numbers (starting with 07 in the UK), and then mapped into another signal that emits `Bool` values.

We are going to go through how all of this works in detail below:


### Creating a signal

The main `Signal` constructor is the following:

```swift
public init(@noescape _ generator: Observer -> Disposable?)
```

So we can create `Signal` a signal by passing a closure that takes an object of type `Observer` and returns a  `Disposable` (more on disposing below). The signal will forward any events send to this `Observer` to all of its subscribers.
If it all seems clear as mud, let's illustrate this with the signal for a counter in the example above:

```swift
func timerSignal() -> Signal<String, NoError> {
    var counter: Int = 1
    
    return Signal { observer in
        
        NSTimer.schedule(repeatInterval: 1.0) { _ in
            observer.sendNext("Counter: \(counter)")
            counter += 1
        }
        return nil
    }
}
```
**NB**: The `schedule:` method is provided by an extension so we can simplify the code example by avoiding the use of selectors. The same could have been achieved without it of course.

Our `timerSignal` will emit events of type `String` and will return no errors. Every time our timer fires (that is every second), we send a new `String` event with the updated count. Now, once this signal is created by invoking the `timerSignal()` method, it will begin emiting next events irrespective of whether anybody is listening to them or not.


### Observing
We've just created our very lovely signal, so we want to know (that is observe) events that come out of it so we can do something about it:

```swift
timerSignal()
    .observeNext { [weak self] next in
        self?.countLabel.text = next
}
```

We're going to observe `Next` events (remember they're of type `String`) and every time we get a new one, we'll update out countLabel text. If you run the demo application, you will see the label changing every second with the updated count. Voila!


## Manipulating signals

So far, we've discussed the Reactive part of FRP. User interaction, networking, timers and so on produce streams of events called signals. We can observe this signals if we're interested in their values. We react to events, because we don't know when they will be happening. The great advantage, is that it's easier for us to avoid polling and have different components of the app work asynchronously of each other. As iOS developers, we're already used to this kind of pattern because we use it with `NSNotificationCenter`. One part of the app may emit a notification (in effect, an event) and another will listen to notifications if relevant. It's exactly the same concept. 

The real power of FRP is its combination with functional programming. Unlike a notification, a signal can be combined with other signals; it can be filtered (like an array); it can be throttled, delayed, mapped, flat-mapped, events can be skipped, values can be transformed into other signals, and so on. Most of these transformations have no side-effects, meaning that the original values are not altered. This is great because we can significantly reduce state, and consequently bugs.

Let's have a look at some of of the possible transformations (for more constructs have a look at [ReactiveCocoa docs](https://github.com/ReactiveCocoa/ReactiveCocoa/tree/master/Documentation))

### Map

Applying `map` to a signal works exactly the same way as it does for arrays: We get a new signal of whatever type `map` has returned. Every time the original signal emits an event, it gets transformed by `map`. Let's take a look at our example, where we convert lower case text typed by the user into uppercase.

```swift
func lowerToUpperSignal() -> SignalProducer<String, NSError> {
    
    return lowercaseTextField
        .rac_textSignal()
        .toSignalProducer()
        .map { next in
            guard let string = next else {
                return ""
            }
            return string.uppercaseString
        }
}
```
**NB**: A `SignalProducer` is a type of signal that doesn't emit events until it's started. We will discuss this further in part two, for now just imagine its the same as a `Signal`.


```swift
lowerToUpperSignal()
    .startWithNext { [weak self] next in
        self?.capsLabel.text = next
}
```

`map` converts our lowercase values into uppercase, so when we listen to the transformed signal we get all uppercase text.


### Filter
We can use `filter` to make sure that certain types of events that fulfill a predicate go through. A new signal is created which emits only events that match that filter. In our example, we're using `filter` to make sure that the text that starts with 07 (a UK phone number) go through. We then use map to convert the `String` event into a `Bool`value of `true`.

```swift
func phoneSignalProducer() -> SignalProducer<Bool, NSError> {
    
    return phoneTextField
        .rac_textSignal()
        .toSignalProducer()
        .filter { next in
            guard let string = next else {
                return false
            }
            return string.hasPrefix("07")
    }
        .map {_ in 
            return true
    }
}
```

We can then start observing the signal:

```swift
phoneSignalProducer()
   .startWithNext { [weak self] next in
        self?.phoneLabel.text = "YES"
}
```

We now start seeing the real power of FRP, given how easy we've combined `filter` and `map` to produce only `Bool` values that we're interested in.


### Throttle
Ocassionally, we want to throttle a signal so we don't get too many events in quick succession. For example, if we're implemented a search, we may not want to fire up loads of network requests. We can throttle a signal, so the number of network requests doesn't end up being too high. 

As a test, we can throttle the signal from the lowercase textfield, so the conversion to uppercase doesn't happen immediately:

```swift       
 lowerToUpperSignal()
    .throttle(0.5, onScheduler: QueueScheduler.mainQueueScheduler)
    .startWithNext { [weak self] next in
        self?.capsLabel.text = next
}
```

It's incredibly simple, just adding a call to `throttle` is all it takes (it's possible to even specify on what thread the scheduling should run).

As a point of comparison, it's worth thinking about how this would be implemented without FRP. At a minimum, we would need a timer and some shared state to check if we're allowed to update the textfield or not.
Here's a very simple implementation just so you can see how much more code we need:

```swift
private var isThrottling = false

private func configureAlternativeToUppercase() {
    lowercaseTextField.addTarget(self,
                                 action: #selector(textFieldDidChange(_:)),
                                 forControlEvents: .EditingChanged)
    
    NSTimer.scheduledTimerWithTimeInterval(0.5,
                                           target: self,
                                           selector: #selector(throttleTimer),
                                           userInfo: nil,
                                           repeats: true)
}

func throttleTimer(timer: NSTimer) {
    isThrottling = false    
}

func textFieldDidChange(textField: UITextField) {
    
    if isThrottling {
        return
    }
    capsLabel.text = textField.text?.uppercaseString
    isThrottling = true
}
```

You can see how much more combersone it is: we end up with various expose public functions (required for selectors to work), access to the Objective-C runtime and the need for shared state which is bug-prone.

## Dealing with memory

Signals are retained by their observers, so even if you don't retain them, they will be kept in memory so long as their observer is alive. There are therefore various ways of disposing of a signal:

### 1. Send a `Completed` event.

This is a nice way of disposing of a signal, if no more data will be sent. We simply send a `Completed` event and the signal will be diposed of. For example, our timer signal could be ended when the count reaches 10:

```swift
func timerSignal() -> Signal<String, NoError> {
    var counter: Int = 1
    
   // Signal must return disposable?
    return Signal { observer in
        
        NSTimer.schedule(repeatInterval: 1.0) { _ in
            observer.sendNext("Counter: \(counter)")
            counter += 1
            
            if counter == 10 {
                observer.sendCompleted()
            }
        }
        return nil
    }
}
```

### 2. We can use the `takeUntil`, `takeWhile` and `take` methods to end the based on a condition.

The easiest is the `take` method, which simply takes the number of events we wish to receive before the signal is ended. The `takeWhile` method takes a predicate, so when that condition is met the signal ends.
Finally, the `takeUntil` method can be hooked to events of another signal, when we receive an next event from it, our signal will be terminated.

Our timer signal will be terminated when the number of characters in a `String` event is 5 or above.

```swift
timerSignal().takeWhile { $0.characters.count < 5 }
```

The timer signal will be terminated after 10 events are received.

```swift
timerSignal().take(10)
```

Our lowerToUpper signal will be terminated when the `ViewController` is dealloc'd as that's when the `WillDeallocSignalProducer` will emit a `Next` event.
            
```swift
lowerToUpperSignal()
    .takeUntil(self.rac_WillDeallocSignalProducer())
    .throttle(0.5, onScheduler: QueueScheduler.mainQueueScheduler)
    .startWithNext { [weak self] next in
        self?.capsLabel.text = next
}
 

```


### 3. Use the returned disposable, and call the `dispose()` method.

This is the least recommended method, and it's usually only used when a request is cancelled by the user for example. However, it's a lot cleaner to use `takeWhile` with a signal that's connected to say a cancel button. 

```swift
phoneSignalProducer()
   .startWithNext { [weak self] next in
        self?.phoneLabel.text = "YES"
}
.dispose()
``` 


## Conclusion
Hopefully, this introduction has given you a basic understanding of FRP and of ReactiveCocoa in particular. Do download the example, and play around with it so you can familiarise yourself with the various concepts discussed here. 
