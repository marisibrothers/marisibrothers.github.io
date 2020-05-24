---
layout: post
title: Documenting your software
date: '2016-01-30T23:00:00.000Z'
author: luciano_marisi
tags: 
permalink: /2016/01/documenting-your-software.html
---

**TL;DR - Document your code now, your colleagues and your future self would thank you for.**

When it comes down to documenting any software I write, I focus on a good README file and well documented interfaces within the codebase. Most of it will be obvious to developers but I believe it is still worth mentioning.

## README File

I find this file to be very important as this will be the starting point to onboard someone onto your code. The more concise and relevant the better since nobody reads long README files. Naturally, it'll grow with the size of the project but as my history teacher used to say, "don't waffle!".

Generally speaking I'll include these topics (if relevant) in a README file :

- A basic summary of the project containing it's purpose
- The relevant environments and/or configurations
- Screenshots of the software component in actions (a picture is worth a thousand words)
- Architecture diagrams for larger projects
- For libraries, an installation guide that explain how to integrate the library to your project
- Project specific examples

For example, the README file of [JSONUtilities](https://github.com/lucianomarisi/JSONUtilities/blob/master/README.md) contains all the relevant information to get started.

## Documenting code (Interface Documentation)

As developers we like to write self-documenting code. This way it's easy to understand what our code does. We only add comments explaining "why" something is done and not "how". However, the interfaces should be documented to explain and define the intention of a class or a method for example. This allows for explanation of intent of what a software component should do while going through the code. For example, to understand why a class was created or what the purpose of a function is. By documenting the interfaces the contracts between software components are clearer and less prone to ambiguity when used or changed in the future. In the context of iOS/Mac development the interfaces and properties can be documented using HeaderDoc-style comments. [VVDocumenter](https://github.com/onevcat/VVDocumenter-Xcode) is great Xcode plugin to help with this as it prefills all the boilerplate part of the comment.

For example a code snippet of [JSONDecoder.swift](https://github.com/lucianomarisi/JSONUtilities/blob/master/JSONUtilities/JSONDecoder.swift) from [JSONUtilities](https://github.com/lucianomarisi/JSONUtilities) clearly explains what the purpose of `Decodable` protocol is:

```swift
/**
 *  Use the Decodable protocol to support nested JSON objects
 */
public protocol Decodable {
  /**
   Creates a instance of struct or class from a JSONDictionary
   
   - parameter jsonDictionary: The JSON dictionary to parse
   
   - throws: Throws a decoding error if decoding failed
   
   - returns: A decoded instance of the type conforming to the protocol
   */
  init(jsonDictionary: JSONDictionary) throws
}
``` 

These comments can then be translated into documentation if you need to by using for example [appledoc](https://github.com/tomaz/appledoc) for Objective-C or [Jazzy](https://github.com/realm/jazzy) for both Swift and Objective-C.