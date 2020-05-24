---
layout: post
title: Separating data from logic in Swift
date: '2015-12-05T16:40:00.005Z'
author: nahuel_marisi
tags:
- swift
permalink: /2015/12/separating-data-from-logic-in-swift.html
---

**TL;DR - Try to separate data from logic for better code reusability. Ideally, data structures and their properties should be immutable.**

We've recently been discussing how we can [design reusable components](http://www.marisibrothers.com/2015/11/designing-reusable-components-part-1.html) and how we can [provide useful estimates](http://www.marisibrothers.com/2015/11/the-art-of-estimation.html) for our work. 

Today, I want discuss a key concept that can help with writing reusable code: *Separating the data from the program's logic*. This is nothing more than implementing the [Separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns). The idea is to illustrate this with the Onboarding component we've discussed in the prior post about designing reusable components. The examples will be in Swift, so while some of the language facilities might be unavailable in different languages, the concepts should still be useful. 


## Identify what is data

The first step to separate the data from the logic is to identify what is data. In our onboarding example, the data is composed of all the attributes that must be customizable (identified in our design phase) for our onboarding to be reusable in other projects. These were:

- Button placement
- Fonts
- Background image
- Animation speed
- Corner radius of body copy box
- Line width of body copy box
- Border colour
- Text aligment (justified, centred)
- Logo image

The logic in this example will mainly boil down to presenting the different onboarding pages and UI elements based on the data provided. User interaction with the onboarding would also be part of the program's logic.



## Modelling the data

Ideally, we want the data to be inmmutable in order to reduce the errors that happen when one part of the program modifies the data affecting another section that didn't expected that data to change.

The first step is therefore to use immutable properties wherever possible. This means using constants instead of variable (`let` instead of `var` in Swift), and copying data rather than passing a reference to it. In Swift, `structs` are ideal for modelling data because they use value-semantics. When a struct is passed to a method or a function, a copy is made. This method can then alter the data without affecting other methods that might have a copy of the struct.

To illustrate this point, we can have a look at the struct used to contain the attributes of our onboarding example: 

```swift
struct FlexiOnboardContent {
    let backgroundImage: UIImage?
    let logoImage: UIImage?
    let title: String
    let text: String
    
    let titleFont: UIFont?
    let titleColor: UIColor?
    let textFont: UIFont?
    let textColor: UIColor?
    let isTextCentered: Bool?
    
    let hasBorder: Bool?
    let borderWith: CGFloat?
    let borderColor: UIColor?
    
    let hasRoundedCorner: Bool?
    
    let action: (() -> (Void))? 
}
```

The `FlexibleOnboardContent` structure can then be passed to the main onboarding class to instantiate a new onboarding component. All of the properties in this example are optionals because none are required.

There might be situations, such as when the data is managed by Core Data, where it's not possible to use structs. In such situations, reducing the parts of the code where data is accessed might be the best option. 

## Conclusion

We've seen through this brief post how we should attempt to separate the program's logic from its data by using inmmutable properties and value-semantics. A process of trial and error might be required in some instances, so it's definately worth using Swift's playground to try out your ideas.


