---
layout: post
title: 5 tips when using Core Data in Swift 2
date: '2016-01-24T23:21:00.001Z'
author: nahuel_marisi
tags:
- swift
permalink: /2016/01/5-tips-when-using-core-data-in-swift-2.html
excerpt: " "
---

# 1. Don't use optionals for required types

When you generate your NSManagedObject subclasses, Xcode will declare all your properties as optionals, even those that you've marked required. This means that when you instantiate a class with required properties in Core Data, everything will work fine until that entity is validated by Core Data, either by saving that entity or when you call validateForInsert(). It's therefore better to remove the optional marker (?) on properties that are required in the Core Data model.

# 2. When using relationships, remember not to use Swift sets

While you can use Swift data types in most cases, one exception (this might change in the future) are relationships. You must still use NSSet for relationships, so you won't get the same level of type checking. If you attempt to use a Swift Set, the relationship won't be persisted nor will it be seen by Core Data as a relationship. I would suggest you use the method validateForInsert() mentioned before to validate your relationships instead.

# 3. Use Swift number types, but be careful with the requirements

You don't need to use NSNumber for number types in Core Data, you can use Swift's own internal data types. What you need to bear in mind is that you must specify the exact type you've used in Core Data. So for example, if you've selected "Integer 32", you must use Int 32 on your Core Data class.

# 4. Use a simple Core Data stack as possible

As you've probably seen, there are plenty of articles telling you how Core Data can be used in a multi-threaded environment; how it's possible to have multiple NSManagedContext objects that are synchronised and so on. I would suggest that you start with a simple Core Data stack that does all of its operations in a single thread. If later on you find that you need a more complex setup, by all means expand your Core Data stack at that point. Naturally, if you know from the start that your app will require multi-threaded Core Data (for example because there's bound to be large downloads) then you can of course start with such a stack. The idea is to avoid premature optimization.


# 5. Don't use inheritance in Core Data Entities

Strictly speaking, this is not a Swift-specific tip. While Core Data is not a mere object relationship mapper, it does tend to be used to persist data using a sqlite database. If your entities inherit from other entities, they will end up sharing a table in the sqllite database. This can lead to serious performance issues once you your database grows to a few thousand entries. It's best to have some duplicated fields, or use inheritance in your NSManagedObject classes but not on the actual Core Data model.