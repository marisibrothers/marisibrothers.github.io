---
layout: post
title: 'Swift UI previews on iOS 12 and below'
date: '2020-06-05'
author: luciano_marisi
tags: 
- swiftui
- swift
permalink: /2020/06/swiftui-previews-on-existing-projects.html
---

This article explains how to use SwiftUI on iOS 12 or below projects. Specifically, to display previews of existing views and view controllers during development.


# Preview a view controller using SwiftUI

The simplest thing to start with would be to display a view controller that represents a screen. For example, if had the following view controller:

```swift
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .lightGray
        let label = UILabel()
        label.text = "Hello world"
        label.backgroundColor = .gray

        view.addSubview(label)
        label.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: view.centerYAnchor),
        ])
    }
}
```

We can implement [`UIViewControllerRepresentable`](https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable) to instatiate a `ViewController` for SwiftUI to use and then implement `PreviewProvider` to make Xcode render it.

```swift
import SwiftUI

// This prevents a runtime crash in case somebody initialises
// this type on a device with a version earlier than on an iOS 13
// as SwiftUI is not available
@available(iOS 13, *)
struct ViewControllerPreview: UIViewControllerRepresentable {
    func makeUIViewController(context _: Context) -> ViewController {
        return ViewController()
    }

    func updateUIViewController(_: ViewController, context _: Context) {}
}

// This is needed since the `some` keyword is only available on
// iOS 13 and above
@available(iOS 13, *)
struct ViewController_Previews: PreviewProvider {
    static var previews: some View {
        ViewControllerPreview()
    }
}
```

This produces the following preview, note that I've selected a iPhone SE (2nd generation) simulator on Xcode:

<p align="center">
   <img src="/assets/images/viewcontroller_preview.png" width="50%"/>
</p>

With a small snippet of code we can see how the `ViewController` looks without having to run the simulator and navigate to that screen. 

**Notes:**
- The project can have a deployment target of iOS 12 or below but an iOS 13 simulator needs to be selected for the preview to shoe. Xcode seems to use the selected simulator to render the preview.
- To prevent runtime crashes, it is convenient to mark all SwiftUI related types with `@available(iOS 13, *)`. This will ensure iOS does not try to load SwiftUI on an iOS 12 device.


# Preview a custom `UIView` using SwiftUI

The next step is to preview a custom view, for example if we had the following:

```swift
class ContentView: UIView {
    init() {
        super.init(frame: .zero)
        setupViews()
    }

    required init?(coder _: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    private func setupViews() {
        let label = UILabel()
        label.text = "I am a ContentView"
        label.backgroundColor = .orange
        label.numberOfLines = 0
        addSubview(label)

        label.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            label.leadingAnchor.constraint(equalTo: leadingAnchor),
            label.trailingAnchor.constraint(equalTo: trailingAnchor),
            label.topAnchor.constraint(equalTo: topAnchor),
            label.bottomAnchor.constraint(equalTo: bottomAnchor),
        ])
    }
}
```

We would then create a preview of the `ContentView` by implementing [`UIViewRepresentable`](https://developer.apple.com/documentation/swiftui/uiviewrepresentable) and `PreviewProvider`.

```swift
@available(iOS 13, *)
struct ContentViewWrapped: UIViewRepresentable {
    func makeUIView(context _: Context) -> UIView {
        return ContentView()
    }

    func updateUIView(_: UIView, context _: Context) {}
}

@available(iOS 13, *)
struct ContentViewWrapped_Previews: PreviewProvider {
    static var previews: some View {
        ContentViewWrapped()
    }
}
```

This results in a preview that stretches to the size of the simulator that is selected.

<p align="center">
   <img src="/assets/images/customview_pin_to_simulator_size.png" width="50%"/>
</p>

To constrains the view to it's intrinsic size we would set it to [fixedSize()](https://developer.apple.com/documentation/swiftui/group/3284808-fixedsize) and set the preview layout to [`sizeThatFits`](https://developer.apple.com/documentation/swiftui/previewlayout/sizethatfits):


```swift
@available(iOS 13, *)
struct ContentViewWrappedFixed_Previews: PreviewProvider {
    static var previews: some View {
        ContentViewWrapped()
            .fixedSize()
            .previewLayout(.sizeThatFits)
    }
}
```

Unfortunately this does not have the intended result:

<p align="center">
   <img src="/assets/images/custom_view_preview_actual.png"/>
</p>

We would expect something similar to this:

<p align="center">
   <img src="/assets/images/custom_view_preview_expected.png"/>
</p>

*Note that the above expected image was created using a `Text` component from SwiftUI:*

```swift
@available(iOS 13, *)
struct Text_Previews: PreviewProvider {
    static var previews: some View {
        Text("I am a ContentView")
            .background(Color.orange)
            .previewLayout(.sizeThatFits)
    }
}
```

*As of the time writing this article it does not look like SwiftUI's `UIViewRepresentable` can calculate the size of a simple `UIView` that uses Auto Layout. Calculating the size manually and overriding [intrinsicContentSize](https://developer.apple.com/documentation/uikit/uiview/1622600-intrinsiccontentsize) SwiftUI renders the view with that size. Therefore, using [fixedSize()](https://developer.apple.com/documentation/swiftui/group/3284808-fixedsize) does not seem to work with custom views using Auto Layout. I guess we can go back to making views like we did on iOS 5 prior to the release of Auto Layout.*

Hopefully, I'm either misunderstanding how SwiftUI works or this is a bug. Otherwise, if it is not possible to wrap `UIView` subclasses using `UIViewRepresentable`, it would make adopting SwiftUI on existing apps more difficult as one could not use the existing custom views with new SwiftUI code.

A way to bypass this problem is to place the custom `UIView` in a view controller and then preview the view controller. This is done by creating a `ViewPreviewer` type that implements `UIViewControllerRepresentable` and adding an extension to `UIView` to have a nicer entry point.

```swift
import SwiftUI

extension UIView {
    @available(iOS 13, *)
    func embedForPreview() -> some View {
        return ViewPreviewer(contentView: self)
    }
}

@available(iOS 13, *)
private struct ViewPreviewer: UIViewControllerRepresentable {
    let contentView: UIView

    func makeUIViewController(context _: Context) -> UIViewController {
        return ViewControllerContainer(view: contentView)
    }

    func updateUIViewController(_: UIViewController, context _: Context) {}
}

private class ViewControllerContainer: UIViewController {
    let container: UIView

    init(view: UIView) {
        container = view
        super.init(nibName: nil, bundle: nil)
    }

    required init?(coder _: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        view.addSubview(container)
        container.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            container.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            container.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            container.leadingAnchor.constraint(greaterThanOrEqualTo: view.leadingAnchor),
            container.topAnchor.constraint(greaterThanOrEqualTo: view.topAnchor),
        ])
    }
}
```

We can then preview the `ContentView` view using:

```swift
@available(iOS 13, *)
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
            .embedForPreview()
    }
}
```

By default the preview is shown in a simulator:

<p align="center">
   <img src="/assets/images/custom_view_in_container_simulator_size.png" width="50%"/>
</p>

To make the preview bounding box smaller a fixed size for the preview can be provided:

```swift
@available(iOS 13, *)
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
            .embedForPreview()
            .previewLayout(.fixed(width: 200, height: 200))
    }
}
```

This results on:

<p align="center">
   <img src="/assets/images/custom_view_in_container_200x200.png"/>
</p>

This is not ideal but at least it allows as to see how the view would layout on different sizes. For example, using `.previewLayout(.fixed(width: 100, height: 100))`

<p align="center">
   <img src="/assets/images/custom_view_in_container_100x100.png"/>
</p>


# Conclusions

I find it useful to have previews of view controller using SwiftUI since this is faster that having to launch the simulator and navigate to that screen. Also, it helps with decoupling view controllers as it needs to be easy to initialise them when implementing `UIViewControllerRepresentable`.

Since this approach is simple and it helps with development while keeping production code the same, this is a handy trick for any codebase that uses Xcode 11.

*I’d like to thank [Nahuel Marisi](http://twitter.com/nmarisi) for reviewing this article.*