---
layout: post
title: 6 tips about adopting Swift in your project
date: '2015-10-15T09:42:00.000+01:00'
author: nahuel_marisi
tags:
- swift
permalink: /2015/10/6-tips-about-adopting-swift-in-your.html
---

We’ve been using the Swift language since the end of last year, when it reached version 1.0. Initially, we included Swift code in existing codebases but we have now moved to using almost exclusively Swift for new projects. The following tips are based on our experience in working with the language.

# 1. Swift is still a fluctuating and changing language

Even though Swift has long reached version 1.0, it’s still a language that’s very much in development. This means that its syntax and structures can change with every new release, breaking existing codebases. Apple provides an automated conversion tool, but it seldom solves all your problems. The example below is from a medium-sized project written in Swift 1.2 and converted to Swift 2.0. Even after the automatic conversion, there are still 48 issues left.  ThSwift is a fluctuating language, so new version can break existing code bases. 

<p align="center">
   <img src="/assets/images/swift_errors.png"/>
</p>

# 2. Get your developers to follow news about the language’s development

Apple is usually good at discussing changes to the language before they’re actually implemented. Getting your developers to follow their official swift blog (https://developer.apple.com/swift/blog/) and look through the documentation of early beta releases. In this way, they’re more likely to be prepared to adapt your codebase when a new version is released by apple.

# 3. Use it for new projects

Swift 2 has recently been released together with iOS 9 and watch os 2. Many features that were missing have been added and the language feels a lot more polished. It definitely makes sense to start using it for new projects. There’s by now plenty of documentation and help available given that it’s been more than a year since the first stable (1.0) version became available. Using it for new projects will also avoid the need to work with legacy objective-c code.

# 4. Avoid rewriting existing codebases

If you have a large, existing project you could start using Swift for new features first. The interface with Objective-C is good and well-established (as Apple uses it for their own frameworks), so try to avoid the temptation to rewrite large chunks of your battle-tested code. Over time more of your project will be in Swift as components that require changes are slowly improved and rewritten. Have a look at the concept of strangling vines if you have a large code base that will need be maintained for several years: http://www.martinfowler.com/bliki/StranglerApplication.html

# 5. Sell it to your clients / managers

In the long-run, Swift will replace Objective-C. There are now four platforms Mac, iOS, Apple Watch and the new Apple TV where Swift can be used. Given that the last two are very new, it makes sense to invest now in using swift there for new projects. Furthermore, Swift is a much safer language than Objective-C, which means less bugs and more time to concentrate on features.

# 6. Take advantage of the playground to prototype ideas

Swift has a powerful ‘playground’, a feature that allows developers to write code and see a preview of the results. This can be used to prototype all sorts of ideas including animations. By using this tool, you can discuss new ideas with your team, including designers, more effectively.
