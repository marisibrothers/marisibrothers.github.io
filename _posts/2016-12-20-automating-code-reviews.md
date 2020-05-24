---
layout: post
title: Automating code reviews (in the context of iOS development)
date: '2016-12-20T18:00:00.000Z'
author: luciano_marisi
tags:
- iOS
- code review
- continuous integration
permalink: /2016/12/automating-code-reviews.html
---

**TL;DR - Run tests, analyse code coverage and enforce consistent code style automatically in pull requests so you can focus on actually reviewing the code**

When it comes to reviewing code in pull requests, I find that it's important to focus on what the code is doing. Over time I found that automating certain processes related to code reviews has helped me do just that. This post is about explaining the three areas I automate in pull requests, how these things can be automated and why it's valuable to do so.

## A few caveats

The tools mentioned all work if your repository is in Github and they are free if the repository is public. I can't say for certain they'll work with other repository hosting services but the concepts are still very much valid.

Note that this is not a step by step guide on how to integrate these tools into your codebase, most things are easy enough that by either looking into a [project](https://github.com/lucianomarisi/JSONUtilities) or doing a quick google search you can figure out how to set them up.

## 1. Running unit tests

The title is pretty self explanatory here. The idea is that on every pull request the unit tests are run so that any broken tests are flagged. This also has the added benefit of checking that the new code does not actually break the build.

The easiest way to run the unit tests on pull requests is using a cloud based continuous integration provider to avoid having to setup any servers. I've always used [Travis CI](https://travis-ci.org) for open source and I cannot recommend it enough. Alternatively, there are [a lot](https://github.com/integrations/feature/continuous-integration) of providers to choose from. Most of these tools will run both the unit tests on the code you push as well as the result of the merged code.

The key reason why this is very valuable is to ensure that broken builds are not merged into your main branch. If you have a reasonably good test suite it also mitigates against introducing bugs when adding new code. Finally, running tests on another system gives more confidence that the setup is independent on the computer it's running on. For example, if the Xcode scheme is not shared, you would be able to build locally but anybody cloning the repository will not be able to run a build.


## 2. Checking code coverage of new changes

Code coverage is the concept of keeping track of what lines are executed and which are not by your unit tests. Technically, you could write a test that executes every line of code in your project meaning the percentage coverage is not very meaningful on its own. A low percentage will flag a lack of unit tests but a large percentage is definitely NOT a guarantee of having a good test suite. 

The percentage code coverage can be global for the project or just for the diff in a pull request. Since Xcode 7 it's quite easy to generate coverage reports, it's only a tick in the scheme. The produced reports are not very useful on their own, it's best to post them to a code coverage service for analysis. Historically, I've always used [Codecov](https://codecov.io/)  but I know people that use [Coveralls](https://coveralls.io/). Both have the same key features and integrate easily with Github, so I'd say choose whichever one you feel most comfortable with. I'm sure there are other tools that accomplish the same if you find neither to be suitable.

These tools comment on the pull request with the global and diff code coverage percentage as well as a detailed report of what lines are hit and which ones are not. I find this very valuable in flagging code paths that I forgot to test. Therefore, before merging a pull request it's possible to make an educated decision whether there is any value in testing any code that is not currently covered.


## 3. Enforcing code style

Most developers have their own styles and arguably there are no right or wrong styles. Enforcing is probably a strong word but generally speaking it's better to have a consistent style throughout the code you work on.

To do this there are [quite a few](https://github.com/integrations/feature/code-quality) options. I have tried [Hound](https://houndci.com) and [Codacy](https://www.codacy.com), they work with different [linting](https://en.wikipedia.org/wiki/Lint_(software)) tools. Linting can mean quite a lot, in this case I'm referring to the process of finding code that doesn't correspond to a predefined style guideline. Personally I prefer Hound because it uses [SwiftLint](https://github.com/realm/SwiftLint) behind the scenes meaning that checking for code style can be done locally and any non-default rules can be defined in the [`swiftlint.yml`](https://github.com/lucianomarisi/JSONUtilities/blob/master/.swiftlint.yml) file.

The main reason to enforce a style guide is so that new developers can be onboarded faster as they know what to expect. Hence, it's easier to focus on what the code is doing instead of how it looks since the code's style will be the same throughout.

## Conclusion

To conclude, it's worth reiterating that these tools prevent you from wasting time looking for and commenting on boring issues allowing you to focus on what the code is actually doing. Setting up these three kind of integrations is very easy. Realistically for somebody with no experience it should not take more than a day.

For open source projects all these tools are free, so there's little reason not to configure them. If you do work on a private repository it's a good idea to invest in them as they will save you plenty of time in the long run.

This [pull request](https://github.com/lucianomarisi/JSONUtilities/pull/9) shows an example in the status check of some of these tools in action. Travis CI has run the unit tests (both on the [commit](https://travis-ci.org/lucianomarisi/JSONUtilities/builds/183541881) and the resulting [merge](https://travis-ci.org/lucianomarisi/JSONUtilities/builds/183541952)), Codecov has analysed the code coverage, and Codacy and Hound have looked for any style issues.