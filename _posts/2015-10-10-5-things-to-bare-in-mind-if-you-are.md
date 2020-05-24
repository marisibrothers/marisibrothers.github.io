---
layout: post
title: 5 things to bear in mind if you are planning to develop apps for tvOS
date: '2015-10-10T18:56:00.000+01:00'
author: nahuel_marisi
tags:
- tvOS
permalink: /2015/10/5-things-to-bare-in-mind-if-you-are.html
---

**TL;DR: We’ve recently been working on our first tvOS app, and while there are many similarities with iOS, there are some crucial differences that is worth taking into account.**

# 1. tvOS offers two distinctive ways for developing software

The first one is TVML, a markup language designed to be accessed from a remote server like a website. The second is the traditional UIKit derived API that is very similar to iOS.

If you’re interested in developing app that will mostly be used to display tv content, then it’s probably worth looking into TVML. Given that you can host your own TVML site, you can update it faster and link it directly to your content. TVML apps usually consist of a set of menus and pages that are linked together. 

If you want to develop a game or any kind of interactive app that goes beyond pages and menus and is less likely to be content-heavy (like Netflix) then using the standard tvOS SDK is the right to go. The rest of the article is primarily focused on this method.

# 2. Pay close attention to the Apple TV remote

The remote control of the Apple TV does not function like a trackpad or like the touch screen of the iPhone. There is no cursor that the user can move about. Instead, a part of tvOS called ‘the focus engine’ decides which element should be in focus depending on the user’s gesture. So in a menu, if you swipe left, chances are that the closer element to the left will now be in focus. When an element is in focus, it is animated to indicate that it’s now selected.

The trackpad’s coordinate system matches that of the screen. When dealing with games it’s worth bearing in mind that the first tap on the trackpad (no matter where the finger is placed) has the same coordinate as the centre of the screen.  Given that the screen is HD, the middle of the trackpad is therefore at (960, 540). As the finger is dragged the x and y coordinate values of the touches will change accordingly. So, for example if the finger is dragged towards the left the x coordinate will be reduced.  It’s important to understand this when designing the control of any game. It’s less of an issue for UIKit apps because you will generally use the focus engine to select elements in the screen. 

# 3. Very limited local storage

You’re only allowed to use 500 kilobytes through NSUserDefaults. This means that unless your storage requirements are minimal, you will have to use iCloud to save your app’s data or roll your own solution. 

# 4. The TV will be watched from the distance

Unlike iOS apps, the TV is used from a distance, so you must make sure that text is readable, graphics are clear, game elements are visible enough. The reality is that experimentation and testing will be necessary to determine what works best from two or more meters away. 

# 5. The Apple TV can only be paired with one Apple remote and a maximum of two third-party bluetooth controllers at the same time 

This has huge implications for multiplayer games. You will have to think hard before you decide that a third-party controller is required for multiple players. Perhaps, a turn-by-turn solution can be considered, depending on your game. You should also bear in mind that your app must work with the Apple TV remote, so in other words it can’t require a third-party remote, or it will be rejected by Apple. 
