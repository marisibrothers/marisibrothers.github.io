---
layout: post
title: Simple styled labels
date: '2016-06-12T20:05:00.000+01:00'
author: luciano_marisi
tags:
- software architecture
- swift
- design patterns
permalink: /2016/06/simple-styled-labels.html
---

**TL;DR - Define the various styles (font, colour, etc) of labels using subclasses**

The apps we develop have text in different styles. These styles are reused across the app, e.g. all titles will be in *Helvetica Neue Bold* size 16. To make things easier, your designer may provide you with a typography stylesheet that defines all the styles used in the which looks something like this (probably with a fair few more combinations):

|                               | **#FFFFFF (white)** | **#000000 (black)** | **#008000 (green)** |
|-------------------------------|---------------------|---------------------|---------------------|
| **Helvetica Neue Bold 16**    |                     | Header1             |                     |
| **Helvetica Neue Regular 16** | Header2_white       | Header2_black       |                     |
| **Helvetica Neue Regular 12** |                     |                     | Body                |

Diving into the code, I generally define the fonts I'm using in a `UIFont` extension (category in Objective-C). For example:

The possible fonts and colour could then be defined in `UIFont` and `UIColor` categories. For example, the `UIFont` extension will look something like this:

```swift
extension UIFont {
  
  static func helveticaNeueBold(fontSize fontSize: CGFloat) -> UIFont {
    return fontWithName("HelveticaNeue-Bold", fontSize: fontSize)
  }
  
  //Other fonts...
  
  static private func fontWithName(fontName: String, fontSize: CGFloat) -> UIFont {
    guard let font = UIFont(name: fontName, size: fontSize) else {
      return UIFont.systemFontOfSize(fontSize)
    }
    return font
  }
  
}
```

Once I have convenience methods defined for my fonts and colours I create a struct for name-spacing purpose where I can collate all the available styles. For the case of Header1 in the previous stylesheet the code would be the following:

```swift
struct TextStyles {
  
  static func header1() -> [String: AnyObject] {
    return [
      NSFontAttributeName: UIFont.helveticaNeueBold(fontSize: 16),
      NSForegroundColorAttributeName: UIColor.blackColor()
    ]
  }
  
  //Other styles....
}
```

In this way it's possible to define all the styles used in the app in a sort-of XML format with no real logic. This example is very simple and could certainly be improved to make it more type safe but this is beyond the scope of this post, check out [TextAttributes](https://github.com/delba/TextAttributes) to get an idea of what I mean.

## Introducing `StyledLabel`

Once all the styles are defines it's time to use them in the app in a scalable and consistent way. For `UILabel`s the approach I take is to create a base class named `StyledLabel` that would need to be subclassed for each separate style. This is a very simple class that is responsible for setting the correct style on the label's text.

```swift
class StyledLabel: UILabel {
  
  override init(frame: CGRect) {
    super.init(frame: frame)
    styleText()
  }
  
  required init?(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)
    styleText()
  }
  
  override var text: String? {
    didSet {
      styleText()
    }
  }
  
  private func styleText() {
    attributedText = NSAttributedString(string: text ?? "", attributes: attributes)
  }
  
  // MARK: To be overridden by subclass
  
  var attributes: [String: AnyObject] {
    return [:]
  }
  
}
```

`StyledLabel` can be subclassed to create a label for each of the styles on the stylesheet. For example, the Header1 style would be:

```swift
final class Header1Label: StyledLabel {
  override var attributes: [String: AnyObject] {
    return TextStyles.header1()
  }
}
```

Now it's possible to use `Header1Label` in xibs, storyboards or in code without having to worry about them having the correct style. Additionally, if the design changes for the Header1 style it can be easily changed by only changing the `header1` function in `TextStyles`.

Putting all together in a playground for a quick test:

```swift
import UIKit
import XCPlayground

let containerView = UIView(frame: CGRect(x: 0, y: 0, width: 100, height: 50))
containerView.backgroundColor = .whiteColor()
XCPlaygroundPage.currentPage.liveView = containerView

let header1Label = Header1Label(frame: CGRect(x: 0, y: 0, width: 100, height: 50))
header1Label.text = "Hello world!"
containerView.addSubview(header1Label)
```

All in all this pattern has some great advantages:

- - It's simple to use and understand
- - The styles are defined clearly and in one place
- - It's easy to update a style affecting all the app
- - It can be used in a similar way for `UITextField`, `UIButton`, etc