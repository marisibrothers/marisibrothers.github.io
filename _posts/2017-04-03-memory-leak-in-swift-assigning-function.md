---
layout: post
title: 'Memory leak in Swift: Assigning a function to a closure'
date: '2017-04-03T18:59:00.000+01:00'
author: luciano_marisi
tags:
- swift
permalink: /2017/04/memory-leak-in-swift-assigning-function.html
---

**TL;DR - Assigning a function to closure property creates a strong reference to the owner of the function potentially creating a retain cycle**

Swift has [first class functions](https://en.wikipedia.org/wiki/First-class_function), meaning that a function is treated like any other object, i.e. it's possible to:

- Pass a function as an argument into another function 
- Return a function as a value from a another function
- Assign a function to a variable 
- Store functions in data structures

I have found one caveat related to this. Let's say you have a view that has a closure property that gets called when a button is pressed.

```swift
class View {
	var onButtonPressed: (()-> Void)?
}
```

This `View` is owned by a `ViewController` and it handles the button being pressed by assigning it's `handleButtonAction` function to the `onButtonPressed` closure property of the `View`.

```swift
class ViewController: UIViewController {
	let customView = View()
	
	init() {
		super.init(nibName: nil, bundle: nil)
		customView.onButtonPressed = handleButtonAction
	}
	
	required init?(coder aDecoder: NSCoder) { fatalError() }
	
	func handleButtonAction() { /* some implementation */ }
}
```

Since Swift allows for functions to be assigned to closures I thought that `customView.onButtonPressed = handleButtonAction` was an elegant way of linking the implementation of the handler, defined in it's own function, to the button action. However, this seemingly harmless code will create a retain cycle. Thanks to Xcode 8 memory debugger it's easy to visualise this:

<p align="center">
   <img src="/assets/images/retain_cycle.png" width="50%"/>
</p>

This happens because when the `handleButtonAction` function is assigned to the `onButtonPressed` closure `self` is implicitly captured since `self` is the owner of `handleButtonAction`. The strong reference to `self` from the `handleButtonAction` function is indicated by the blue arrow.

## Unit testing the retain cycle

While finding a solution to break this cycle it's good to have a unit test that proves this. The following test will fail if the `ViewController` has a retain cycle:

```swift
func testViewControllerNotRetained() {
	// Create two variables for the view controller,
	// one strong and one weak
	var sut: ViewController? = ViewController()
	weak var weakSut = sut
	
	// Nilling out the strong reference should release the object,
	// making the weak reference also nil
	sut = nil
	XCTAssertNil(weakSut)
}
```

## Breaking the cycle

A solution to this problem can be to assign a new block that captures `self` weakly and call the function inside it using the `weak self`:

```swift
view.onButtonPressed = { [weak self] in
	self?.handleButtonAction()
}
```

In this case we can go for `unowned` since the `View` will not exist beyond the lifecycle of the `ViewController`:

```swift
view.onButtonPressed = { [unowned self] in
	self.handleButtonAction()
}
```

This solution is certainly less elegant than the original direct assignment. This is why it's so easy to forget that assigning a function retains the owner of that function potentially creating a retain cycle.

*I’d like to thank [Nahuel Marisi](http://twitter.com/nmarisi) and [Neil Horton](http://twitter.com/Neil3079) for reviewing this article.*