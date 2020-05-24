---
layout: post
title: Lazy collections
date: '2017-05-16T13:35:00.000+01:00'
author: nahuel_marisi
tags:
- iOS
- swift
- functional programming
permalink: /2017/05/lazy-collections.html
---

**TL;DR - The lazy evaluation of collections can improve the performance of your app when working with large collections.**

Swift introduced the posibility of lazy evaluation in our code. While many people seem to be aware of lazy variables, I find fewer users of lazy collections. In this brief article, we'll look at how we can use lazy evaluation with collections, which can bring performance benefits when dealing with particularly large collections.


## Lazy map
Let's have a look at what happens when we ran `map` on a standard array.

```swift
let array = Array<Int>(0..<1000)
let mappedArray = array.map { (value: Int) -> Int in
    print("incrementing: \(value)")
    return value + 1
}

print("First element mapped: \(mappedArray[5])")
```

In here, map will apply our closure to every single element of the array, even though we're only interested in the fifth one. If you run this code in the Playground, the output should be something like:

```
incrementing: 0
incrementing: 1
incrementing: 2
incrementing: 3
incrementing: 4
    ...
incrementing: 999
First element mapped: 6   
```

The larger the array is, and the more computationally intensive your closure is, the more pointless work you end up doing if you're not interested in all of the element of this array.

### Using a lazy collection

Every collection provides the `lazy` property that gives us a "view" into this collection, that lazily provides higher order operations like `map` and `filter`. The actual type of the `lazy` property is `LazyMapRandomAccessCollection`. Let's go back to our original example, but use `lazy` instead this time:

```swift
let array = Array<Int>(0..<1000)
let mappedArray = array.lazy.map { (value: Int) -> Int in
    print("incrementing: \(value)")
    return value + 1
}

print("First element mapped: \(mappedArray[5])")
```

Output:

``` 
incrementing: 5
First element mapped: 6
```

This time, map is only applied to the element we're interested (fifth), saving us a favolous amount of pointless work.

### Filter

Just as you can use `map` with lazy collections, you can also use `filter`. Let's have a look at a quick example:

```swift
let array = Array<Int>(0..<1000)
let filteredArray = array.lazy.filter {
    print("filtering: \($0)")
    return $0 < 10
}

print("first filtered result: \(filteredArray.first!)")
```

Output:

```
filtering: 0
first filtered result: 0
```

As we see, filtering only runs once (for the first result), instead of the 1000 times it would run if we were not using a lazy collection. It's worth noting that filter returns `LazyFilterBidirectionalCollection`, which unlike its random access counterpart, does not accept an integer as a subscript. 

#### Prefix

A lazy filter can easily be combined with the `prefix` function to limit the number of results filter returns. It's often the case that we're only interested in the first occurence of an object in a collection. By using evaluation and `prefix` we can stop as soon as we find the first occurence. Without using a lazy collection, filter would still go through the whole collection. 

```swift
let array = ["green", "blue", "red", "orange", "green", " purple", "red"]
let filteredSequence: AnySequence = array.lazy.filter {
    print("filtering: \($0)")
    return ($0 == "red")
}.prefix(1)

print(Array(filteredSequence))
```

Output:

```
filtering: green
filtering: blue
filtering: red
["red"]
```

Note here that we're making sure that our collection is evaluated as a sequence. There are two `prefix` version. The collection one which returns a `BiDirectionalSlice` and the sequence one which returns `AnySequence`. Only the sequence version evaluates lazy, while the collection one will iterate through the whole collection. The drawback of using a sequence is that we lose the use of subscripts. 