---
layout: post
title: Sharing your software components
date: '2015-10-03T18:58:00.000+01:00'
author: luciano_marisi
tags: 
permalink: https://www.marisibrothers.com/2015/09/sharing-your-software-components-tldr.html
---

**TL;DR - It is never been easier to share your code either publicly or internally within your company and it almost always a good idea to do so. So start sharing some of your software today! The post is refers to iOS development.**

Recently I have been making internal SDKs to be used within a variety of client app. This has got me thinking about the need to make the APIs as flexible and as future-proof as possible as well as to have a library that can be easily integrated within a few days within apps that older than 2 years. Within the iOS community we use open source libraries very often in our projects or sometimes to get ideas on how other developer have achieved a specific task. However, a lot of developers generally don't share their software because of various reasons. I think it is generally a good idea to do so either interlly within your company to be used by other projecst or publicly to be used by anyone.

## What do you need to release your code

Ideally it is best not to just make the code available on Github but to document it as well. This means that whoever is looking for something similar to what you have written does not have to go through your code to figure out if it is relevant to them because let's face it they probably won't.

It is easy to dump your classes on Github without any documentation but it is probably not very useful to anybody including your future self. In my opinion, you should cover these areas in order of priority according to the time you have:

1. Documentation
	- README file
		- I have seen many very large projects (private and public) that have no README file whatsoever, which are very hard to understand after 2+ years of development.
	- Documented interfaces (i.e. header files in Objective-C).
	- Images to show you component in action if relevant.
2. Example project to show your software in a
3. How to add your code to any project, for example:
	- Which files to add directly (The lazy way, currently all my code available on Github is done this way.)
	- Support for Cocoapods (Full disclosure, I have not yet released a public cocoapods yet but I have release many private ones, so I have some understanding of most of the process.(
	- Support for Git submodules
4. Units tests are also useful to keep your features in your project from breaking in the future.
5. Continuous integration to give you confidence that neither the build nor the tests are broken when you update the project.

## Why would you share your components

I have thought of a few reasons (this list not exhaustive) as to why you should share your components.

- Better software architecture
	- Helps you understand more about what your software should do and to make sure that it is independent of the rest of the project. This comes because evident when you move your code to an example project and understand all the different dependencies involved.
	- Encourages increase quality and focus on stability since your code will be used by multiple projects
- Can be easily reused on your own projects later
- Can be used by other people
- Other developers can comment and improve that component/library
- If you are looking for work and you release some of your code publicly, prospective employers can look at your code to see how you work and have a better idea if you are the right fit for them.


## Final words

Naturally, this is a very subjective post and my opinion may be different from yours. Anyways, if you don't have time to open source your components now, try to build them in a way that the are easy to open source in the future. This way your program would more modular and more maintainable in the future.

I can't say all the components I've made public follow all the ideas described on this post since some of them are reasonably old or that I shared a lot of the code I have written. However, most of these ideas are simple and very valuable. As of when this post was written I've shared the following components that can be found on the [TechBrewers Github Page](https://github.com/techbrewers).