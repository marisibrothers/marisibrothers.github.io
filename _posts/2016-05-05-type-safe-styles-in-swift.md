---
layout: post
title: Type-safe styles in Swift
date: '2016-05-05T22:28:00.000+01:00'
author: luciano_marisi
tags:
- software architecture
- swift
- design patterns
permalink: /2016/05/type-safe-styles-in-swift.html
---

**TL;DR - By leveraging generics it's possible to create type-safe state values for the different states of a view**

Views in a application will be styled and branded and looking slightly differently depending on the state they are in. For example, for a button that is disabled the label may be greyed out and the shadow removed. Sometimes even the font could change when the state changes. We can leverage generics in Swift to encapsulate this concept.

The first thing to do is to create a generic struct that will encapsulate the values for the different states. Note that this struct will be specialised with anything relevant for a state, e.g. `UIColor`, `UIFont`, etc. In the following struct I've only defined 3 states, *normal*, *highlighted* and *disabled* but there could as many or as few as it's needed since they are optional.

```swift
struct StateValue<T> {
  
  private let normal: T?
  private let highlighted: T?
  private let disabled: T?
  
  init(normal: T? = nil, highlighted: T? = nil, disabled: T? = nil) {
    self.normal = normal
    self.highlighted = highlighted
    self.disabled = disabled
  }
  
}
```

To be able to improve how the `StateValue` struct is used I created an enum that represents some of the states a view could be in. This list matches the previous states thought it doesn't necessarily have to. Again, it's certainly not an exhaustive list, it only covers the relevant states for this example.

```swift
enum ViewState {
  case Normal
  case Highlighted
  case Disabled
}
```

We can then extend the `StateValue` so that we can access the value for a specific `ViewState`. I have used a `subscript` for this because it provides a cleared interface than a function.

```swift
extension StateValue {
  
  subscript (state: ViewState) -> T? {
    switch state {
    case .Normal:
      return normal
    case .Highlighted:
      return highlighted
    case .Disabled:
      return disabled
    }
  }
  
}
```

Now that all the building blocks are defined we can use them to extend any views we would want. For example, `UIButton` could be extended to have a method that will set the colour of the title for all states.

```swift
extension UIButton {
  
  func new_setTitleColor(stateValue: StateValue<UIColor>) {
    setTitleColor(stateValue[.Normal], forState: .Normal)
    setTitleColor(stateValue[.Highlighted], forState: .Highlighted)
    setTitleColor(stateValue[.Disabled], forState: .Disabled)
  }
  
}
```

Putting all this together into a playground and using the new interactive playgrounds for the button it's possible to show this in action.

```swift
import UIKit
import XCPlayground

class Target : NSObject {
  func action() {
    print("Button pressed!")
  }
}

let containerView = UIView(frame: CGRect(x: 0, y: 0, width: 200, height: 200))
containerView.backgroundColor = .whiteColor()
XCPlaygroundPage.currentPage.liveView = containerView
let target = Target()

let button = UIButton(frame: CGRect(x: 0, y: 0, width: 50, height: 50))
button.backgroundColor = .lightGrayColor()
button.setTitle("Some button", forState: .Normal)
button.addTarget(target, action: #selector(Target.action), forControlEvents: .TouchUpInside)
containerView.addSubview(button)

let buttonState: StateValue<UIColor> = StateValue(normal: .purpleColor(), highlighted: .greenColor(), disabled: .redColor())
button.new_setTitleColor(buttonState)
// button.enabled = false // This would make the button red
```

The previous example is one of the ways the `StateValue` can be used. Equally this could be used for encapsulating the themes of different types in your application. This is a lot clearer with an example. Let's say we have an `Vehicle` enum:

```swift
enum Vehicle {
  case Car
  case Airplane
}
```

Let's say in our application the button for a `Car` looks different than the button from a `Airplane` and they are both displayed in some form of dynamic view (e.g. a `UITableView` of many `Car`s and `Airplane`s). To style each of the buttons of the cell we could extend `Vehicle` using a new `Stylable` protocol and define the `StateValue` for each of the `Vehicle`. Note that the `Stylable` protocol is not strictly needed, it's just a way to have named extensions that are clear to read and make our code more generic and reusable.

```swift
protocol Stylable {
  var buttonColor: StateValue<UIColor> { get }
}

extension Vehicle : Stylable {
  var buttonColor: StateValue<UIColor>  {
    switch self {
    case .Car:
      return StateValue(normal: .yellowColor(), highlighted: .blueColor(), disabled: .grayColor())
    case .Airplane:
      return StateValue(normal: .purpleColor(), highlighted: .greenColor(), disabled: .redColor())
    }
  }
}
```

The `buttonColor` on a `Vehicle` could be used in the following way (Bare in mind this is quick example to show the concept in action). 

```swift
let vehicleButton = UIButton(frame: CGRect(x: 0, y: 0, width: 50, height: 50))

func setTitleColorForStylableItem(stylable: Stylable) {
  vehicleButton.new_setTitleColor(stylable.buttonColor)
}

let car = Vehicle.Car
setTitleColorForStylableItem(car)
```

All in all, if you have a lot of styling in your app for the various states and types of object using a generic `StateValue` struct is a great way to encapsulate all this logic. One of the advantages is that because it's generic it can be used for anything relevant to your style as mentioned originally; `UIColor`s, `UIFont`s, `UIImage`, etc. I believe this is a great pattern that can be extended in way that will suit your needs very easily.
