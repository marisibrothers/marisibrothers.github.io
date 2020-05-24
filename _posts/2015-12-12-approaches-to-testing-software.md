---
layout: post
title: Approaches to Testing Software
date: '2015-12-12T18:00:00.000Z'
author: luciano_marisi
tags: 
permalink: /2015/12/approaches-to-testing-software.html
---

**TL;DR - Knowing the different ways and strategies to test software will give you the ability to define your approach depending on your project limitations**

Over the last few weeks we've been exploring less technical topics around software development ([Designing reusable components](/2015/11/designing-reusable-components-part-1.html), [The Art of Estimation](/2015/11/the-art-of-estimation.html) and [Separating data from logic in Swift](/2015/12/separating-data-from-logic-in-swift.html)). In this article I discuss the various approaches I take around testing software. I'll focus on testing of mobile applications, specifically iOS apps but most concepts are applicable to other software.

When I think of testing an app 3 approaches come to mind:
 
- Basic manual testing
- Automated testing
- Exploratory testing

## Basic Manual Testing

The basic and most common testing carried out in software is manual testing. Specifically, this involves a person going through the app manually tapping buttons, filling in textfields and so forth.

To make manual testing work it's important to define very clearly test scenarios. This way any other tester joining the project can follow these test cases to ensure that all the edge-cases known are tested every time. More often than not these test cases are carried out by a click engineer. Someone that will blindly follow a set of instructions tapping buttons without much thought.

Blindly following test cases is of limited value. The tester testing should understand the app in a way that when a bug is found, they can provide insights and hypothesis on the underlining cause of the issue.

## Automated Testing

Manual testing will work on projects that have a lower budget and focus on short term goals. However, it'll be less valuable in the long term as the time taken to manually complete all the test cases is always the same. This is where automation testing is useful since there is a high setup time/cost to write the automated tests but running them is very fast.

The value of automation is sometimes misinterpreted specially when people rely on test coverage percentage as a measure of testing quality or they have a blind obsession with it. Some aspects can be tested using automation and some others can't, and so should be manually tested.

There are some of the types of automated tests I have written:

- Logical unit tests
- Performance unit tests, this one is tricky to test because most of the times we run out test-suite in our CI server or development laptop but it is key this is run on the slowest of the devices that your app is meant to run
- Snapshot tests, which are useful to ensure the UI is consistently the same
- Sensor related tests (e.g. gps location, accelerometer, gyroscope, barometer and so forth), where mocking of the signals is necessary
- UI tests, which ensure the flow between screen does not break
- Server integration tests
- Static content validation tests (Check spelling of content, check all fields expected for the content exist and are of the expected format)

This list is not exhaustive but it will give you an indication of what is possible.

## Exploratory Testing

The real value of automation tests is that it allows for time spent doing exploratory testing. This literally involves using the app in different ways as user would. By doing this it'll be possible to catch edge-cases that were not found by previous manual and/or automated tests.

This is where the human value comes into play. We make apps for people. Therefore, it's best if other people use the app to pick up unexpected and/or undesired behaviour.

## Conclusion

Generally speaking it's best to automate the monotonous part of a an application where it's time efficient to do so. Basic manual testing should be left to cases where automating something would be very time consuming. This is very valuable in the long term as regression testing before a release is reduced. Thus, the focus can be on a more real-life exploratory testing approach.

The testing strategy depends very much on the team developing the software and their limitations (skills, time and so forth). Hopefully, after reading this article you'll be able to make more educated choices about how you test your software.