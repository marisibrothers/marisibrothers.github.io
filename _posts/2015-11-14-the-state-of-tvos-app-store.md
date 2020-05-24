---
layout: post
title: The state of the tvOS App Store
date: '2015-11-14T22:00:00.000Z'
author: luciano_marisi
tags:
- tvOS
permalink: /2015/11/the-state-of-tvos-app-store.html
---

**TL;DR - The tvOS App Store is a bit too light on features and discoverability is very poor meaning that it's very unlikely that even your mother will download your app**

You can look at the previous tvOS articles here:

- [5 things to bear in mind if you are planning to develop apps for TvOS](/2015/10/5-things-to-bare-in-mind-if-you-are.html)
- [Interacting with the new Apple TV remote](/2015/10/interacting-with-new-apple-tv-remote.html)
- [Storing your data on tvOS, part 1](/2015/10/storing-your-data-on-tvos.html)
- [On-Demand Resources on tvOS](/2015/10/on-demand-resources-on-tvos.html)
- [Storing your data on tvOS, part 2: CloudKit](/2015/11/storing-your-data-on-tvos-part-2.html)

## Discoverability

After uploading two very simple no thrills games [Helicopter](#Helicopter) and [Ping Pong Arcade](#PingPongArcade) for tvOS and looking into the tvOS App Store for 2 weeks we have found that discoverability is very poor in its current state. 

As a user it's hard to find relevant apps to you and as a developer it's hard to target specific users to your app. Currently depending on which country you are on you'll see between 3 and 5 tabs; Featured, Purchased and Search, and optionally depending on your country Top Charts and Categories. For example, the UK store does not currently have a categories section.

<p align="center">
   <img src="/assets/images/tvOS_app_store_UK_top_charts.png"/>
</p>

This will get better as Apple is adding various category sections and rankings as more apps come to the store. This is slowly being introduced because of the lack of tvOS apps out there. If you feel like exploring what tvOS apps are out there, you are out of luck.

Searching the tvOS App Store is quite a painful experience (quite literally). Siri dictation cannot be used for searching and it does not look like 3rd party external keyboards are [compatible](https://support.apple.com/en-us/HT202670) with the latest Apple TV. Currently, the only way to search is by using the Apple TV Remote with the displayed keyboard. This is indeed an annoying limitation of the Apple TV ([Apple Support Communities](https://discussions.apple.com/thread/7314087), [MacRumors forum](http://forums.macrumors.com/threads/pairing-apple-keyboard-with-apple-tv-gen-4.1933718/)). On the bright side given the lack of apps in the store only typing a few letters will *generally* find you what you need.

<p align="center">
   <img src="/assets/images/heli_search.png"/>
</p>

Given the current state, if you want to get a descent number of downloads (either paid or free) you need to be featured, so users can easily find your app.

#### Featured section (UK store)

<p align="center">
   <img src="/assets/images/tvOS_app_store_UK_featured.png"/>
</p>

iOS and tvOS apps can share the same bundle ID, meaning that a user that downloads your app on the iOS App Store will see it on the purchased section of the tvOS App Store. Depending on your business model it may make sense to have the same bundle ID for your iOS and tvOS. This way there is a better chance you'll get users downloading your tvOS app since it will be more visible to them as there is a dedicated **Purchased** section.

## Limited deep linking to your app

As far as I know, the only way to link to your app is from another tvOS App. For example:

```swift
if let appStoreURL = NSURL(string: "https://itunes.apple.com/uk/app/helicopter/id1047310922?mt=8") {
      UIApplication.sharedApplication().openURL(appStoreURL)
}
```

Within iOS there are numerous ways of doing this; links on websites, mobile ads, emails, SMS and so forth. At the moment it's hard to picture how deep linking would work on the Apple TV but I'm sure this will be solved in the coming months since tvOS is still very new.

## App previews are limited

After a great adventure, a user has finally found your app (whoohoo!). However, this is only half the battle won, now you have to do everything in your power to convince them to actually download your app. Even though the design of an app page is nice(-ish), it's interesting that there are no app preview videos. This feels like something that is distinctly missing, specially considering this feature is available in the iOS App Store.

### Helicopter

<p align="center">
   <img src="/assets/images/helicopter_app_preview.png"/>
</p>

### Ping Pong Arcade

<p align="center">
   <img src="/assets/images/ping_pong_arcade_app_preview.png"/>
</p>

Only a star rating from 1 to 5 is allowed, no actual reviews. This does make sense when you understand how painful it's to write using the Apple TV remote. However, as a user it's hard to be convinced to download an app after seeing just a star rating. To be honest even if the user could write reviews, there is a pretty good chance that they will end up writing something short using monosyllabic words.

At the moment there are no analytics to know how many people actually reach your app preview page as it's the case of the iOS App Store. This makes it very hard to know if people can't find your app or they find it but don't download it.


## Conclusion

The tvOS store feels very limited, frankly it looks like it's a beta version. It's hard to find the apps you are looking for and explore what is out there. Should you invest time/money in a tvOS app? If you intend to make money out of it and you don't have a huge (and I do mean huge) pile of cash for marketing then probably hold fire for now until the tvOS App Store develops into something more user friendly.

If you are not looking to make money, you should absolutely develop for tvOS. The development is very similar to iOS and it's a good learning experience. The platform is different enough that you will think about UI/UX in a different way. Having been developing for tvOS since September I can see that the Apple TV has a huge potential but currently only big players can reach users given the current state of the tvOS Store.
