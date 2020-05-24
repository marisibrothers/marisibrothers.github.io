---
layout: post
title: Learning a new programming language today
date: '2017-09-20T20:20:00.000+01:00'
author: luciano_marisi
tags:
- software development
- learning
permalink: /2017/09/learning-new-programming-language.html
---

**TL;DR - Understand the core concepts of a language and then alternate between learning details and putting them into practice**

A language is a means to an end. Quickly picking up programming languages allows developers to build anything. I've recently been learning Java to contribute to a [Spring](https://spring.io/) backend. 
I've found that once the foundations of the language have been learned, a great way of learning is to develop a process of cycling through learning details, then putting them into practice. 

<p align="center">
   <img src="/assets/images/learn_and_try.png"/>
</p>

## Getting started - Understand core principles

The first thing to do is understand the core concepts of the language and the main use cases. If you can, get an experienced engineer to mentor you. This way you'll cover the mundane setup stuff and be running code in no time. Spend about a day or two here, enough time to know everything to get you started but not too much that you end up bored and lose motivation.

Ask yourself the following questions:

- What tools do developers generally use and why?
	- A good starting point is to ask what's the most appropriate text editor / IDE to use. 
	- Understand why developers use these tools and what others exist but they don't use. Remember that *if all you have is a hammer, everything looks like a nail*.
- What's the [type system](https://en.wikipedia.org/wiki/Programming_language#Type_system)?
	- Is it [static or dynamic](https://en.wikipedia.org/wiki/Programming_language#Static_versus_dynamic_typing)?
	- Is it [strongly or weakly typed](https://en.wikipedia.org/wiki/Programming_language#Weak_and_strong_typing)?
- What's the [programming paradigm](https://en.wikipedia.org/wiki/Programming_paradigm)? 
	- Is it object-oriented, functional, etc?
	- Is it limited to a specific paradigm or does it support numerous? It's common for a language to support [multiple paradigms](https://en.wikipedia.org/wiki/Programming_paradigm#Multi-paradigm)
- Is it [compiled or interpreted](https://en.wikipedia.org/wiki/Compiler#Compiled_versus_interpreted_languages)?
	- Sometimes the division is not so clear, see [Just-in-time compilation](https://en.wikipedia.org/wiki/Just-in-time_compilation) and [Bytecode compilation](https://en.wikipedia.org/wiki/Bytecode)

Most of the above high-level questions don't have an absolute answer which is why exploring these questions and answers is key. This will allow you to establish strong foundations, on which to learn the details.


## Learn the details

At this point you'll hopefully have some vague idea of what the language does and how to execute some code. So now it's time to get to grips with how to write it.

- Learn some syntax, for example:
	- How are variables and functions declared?
	- What are the available [control flows](https://en.wikipedia.org/wiki/Control_flow)? e.g. if statements, for loops
	- What are the core data structures and how do you create them? (e.g. arrays and dictionaries)
- What kind of features does it have? For example:
	- Does it have [reference](https://en.wikipedia.org/wiki/Reference_type) and [value](https://en.wikipedia.org/wiki/Value_type) types? And how are they used?
	- Can you define interfaces/protocols?
	- How do you write concurrent code?
	- If it's strongly typed, does it have generics?
- What's available in the language's [standard library](https://en.wikipedia.org/wiki/Standard_library)?
- What are the main conventions people follow?
	- Check a code style guide, [for](https://google.github.io/styleguide/javaguide.html) [example](https://github.com/github/swift-style-guide)
	- Do developers rely on 3rd party libraries? If so what the most common ones? For example, I've found that modern Java codebases use [RxJava](https://github.com/ReactiveX/RxJava)

## Dive in head first

Theory is great, but we learn much more through [experience](https://en.wikipedia.org/wiki/Experiential_learning). Nobody learns how to ride a bike by reading a book about it. Try out one of the following in order to gain experience:

1. Contribute to an existing project at your workplace, [pair](https://en.wikipedia.org/wiki/Pair_programming) with another developer if you can.
	- I've found that going into an established codebase where good practices are followed has been awesome. It'll be easier to add new small features by extending existing code or by following existing patterns.
	- Code reviews by other developers will help point out issues quickly.
2. Contribute to an existing open source project. It's common for projects to mark [issues](https://github.com/fastlane/fastlane/issues?q=is%3Aopen+is%3Aissue+label%3A%22complexity%3A+you+can+do+this%22) that outsiders can pick up, this is a great place to start.
3. Create your own project. Find something you're keen on creating and build it with the language you're learning.

Lastly, stay out of your [comfort zone](https://en.wikipedia.org/wiki/Comfort_zone) and push yourself to constantly learn.

*I’d like to thank [Emily Woods](http://twitter.com/emily_m_woods) for reviewing this article.*