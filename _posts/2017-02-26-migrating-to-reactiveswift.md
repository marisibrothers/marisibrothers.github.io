---
layout: post
title: Migrating from ReactiveCocoa to ReactiveSwift
date: '2017-02-26T17:08:00.001Z'
author: nahuel_marisi
tags: 
permalink: /2017/02/migrating-to-reactiveswift.html
---

**TL;DR - ReactiveSwift is the first version of ReactiveCocoa that fully embraces Swift 3. There are various API and functional changes introduced in this major release.**

ReactiveSwift is a major release of what has until now been known as ReactiveCocoa, a popular reactive functional programming framework for iOS / OS X. The are two major changes introduced by ReactiveSwift: Firstly a full migration to Swift 3, including changing the API to match the new naming conventions. Secondly, UI bindings have been stripped out into a new framework, which is now called ReactiveCocoa.


## Matching swift 3 API design

The most obvious change in ReactiveSwift is the new Swift-3-esque APIs. It's quite clear that they have worked hard to migrate all APIs to the new syntax. Below I list the most commonly used, however you can see all changes by looking at the deprecated list [here](https://github.com/ReactiveCocoa/ReactiveSwift/blob/master/Sources/Deprecations%2BRemovals.swift).


### Signal / SignalProducer

| Old Method name  | New method name | 
|---|---|---|
|`take`   |`take(first:)`   | 
|`takeUntil:`   |`take(until:)` |
|`takeWhile`   |`take(while:)`| 
|`observeNext:`|`observeValues:` / `observeResult:`[^1]|
|`startWithNext:`|`startWithResult:` / `startWithValues`| 
|`observeOn`|`observe(on:)`|
|`.throttle(<time>, onScheduler:)` | `throttle(<TimeInterval>, on: )` [^2]|

[^1]: If the signal can send errors, observing result will return either an error or a value
[^2]: e.g.: `.throttle(TimeInterval(0.5), on: QueueScheduler.main)`

### Observer
| Old Method name  | New method name   | 
|---|---|
|`sendNext`|`send(value:)`|
|`sendError:`|`send(error:)` |


### Scheduler
| Old Method name  | New method name   |
|---|---|
|`QueueScheduler.mainQueueScheduler`| `QueueScheduler.main`
  
The method names for  `map`, `filter` and `flatMap` have remained the same.

### Notifications

In ReactiveCocoa, it was possible to get a signal from notifications using the `.rac_notifications` property of the notification center. This has now been moved to the the `reactive.notifications` property.

Before ReactiveSwift:

```swift
NSNotificationCenter
	.defaultCenter()
	.rac_notifications(MyNotificationName)
	.observeOn(UIScheduler()
	.startWithNext() {[weak self] notification in
   		// do something with notification
}        
```

In ReactiveSwift:

```swift
NotificationCenter.default
	.reactive.notifications(forName: MyNotificationName)
	.observe(on: UIScheduler())
	.observeValues{[weak self] _ in
   		// do something.
   }
```

### Memory management of signals and signal producers

When disposing of a `Signal` before ReactiveSwift it was necessary to either send a terminating event, use one of the `take:` methods or dispose of the `Signal`'s disposable.
As the ReactiveCocoa 5 documentation puts it:
> A signal created with `Signal.init` is kept alive until the generator closure releases the observer argument. 

This is no longer the case in ReactiveSwift, as the documentation states: 
>[...] a Signal retains itself as long as there is still an active observer.
In other words, if a Signal is neither publicly retained nor being observed, it would dispose of the signal resources silently.

This might be a subtle difference, but it does mean for example that if a signal is no longer retained by the view controller, and nothing is observing it, then we can be sure that it will be disposed off automatically.

I would still prefer to make the memory management more explicit, for code clarity primarily. It's still possible (and recommended) to use the set of `take` methods to manager memory, as well as directly disposing the signal or sending a `completed` event to it. 

ReactiveSwift introduces a new type, `Lifetime` that can make memory management a little easier when working with multiple `Signal`s or `SignalProducer`s. When dealing with signals, it's important to think about how long should the observation of events last. For example, if we're observing a signal to update a UI component, say a `UILabel`, it makes sense to stop that observation if the label is no longer on screen.
A `Lifetime` is simply an object that can be used to notify a `Signal` or `SignalProducer` we no longer want to observe their events. We can set up a `Lifetime` easily like this:

```swift
private let token = Lifetime.Token()
        public var lifetime: Lifetime {
            return Lifetime(token)
    }
```

We can then use the `take(during:)` function to let the signal know that the object being deallocated will no longer be observing it:

```swift
NotificationCenter.default
	.reactive.notifications(forName: MyNotificationName)
	.observe(on: UIScheduler())
	.take(during: lifetime)
	.observeValues{[weak self] _ in
   		// do something.
   }
```

For `Lifetime` to work properly, we need to keep a strong reference to its token. It's crucial that only one strong reference ever exists for the token. When the object that holds the reference to the token (say a view controller) is deallocated, so will the lifetime, and the signal will know that this particular observation is complete.

### ReactiveCocoa UI extensions

One of the major changes in ReactiveSwift is that all the UI extensions have been removed and a new framework called ReactiveCocoa (the name is of course confusing, as the whole project used to be called that) has been created that contains just the UI extensions. This does make the code much clearer as well as allowing for a leaner application if you're uninterested in UI bindings. I plan to make a more detailed post about the new `ReactiveCocoa` framework later on. For now, suffice it to say that UI objects (such as `UITextField`s, `UIButton`s, and so on have a property called `.reactive` that exposes signals with the various events. For example the `UITextField` has a `.reactive.continuousText` signal, which sends an event with the textfield's text every time it changes. In the example below, the text of a `UITextField` is converted to upper case.


```swift
lowercaseTextField
            .reactive.continuousTextValues
            .map { (text) -> String in
                guard let string = text else {
                    return ""
                }
                print(string.uppercased())
            }
```

