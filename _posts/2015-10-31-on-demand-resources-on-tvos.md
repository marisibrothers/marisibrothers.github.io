---
layout: post
title: On-Demand Resources on tvOS
date: '2015-10-31T16:55:00.000Z'
author: luciano_marisi
tags:
- tvOS
permalink: /2015/10/on-demand-resources-on-tvos.html
---

**TL;DR - tvOS app bundles are limited to 200MB and there is no storage on the Apple TV, so if your apps requires more than 200MB of static assets the option is to use On-Demand Resources**

iOS 9 introduced [on-demand resources](https://developer.apple.com/library/prerelease/ios/documentation/FileManagement/Conceptual/On_Demand_Resources_Guide/) (ODR). This is useful on iOS to reduce the download time of your application so that the user can start using your app quicker and so that the app can be downloaded on a mobile network connection. These benefits are nice things to have but generally not key requirements for an iOS app MVP.

However, on tvOS the limit per app is 200MB (compared to 2GB on iOS) meaning that there is a good chance that you will need to use this feature if your app needs many resources as there is no way to persist large amounts of data on the Apple TV as explained [previously](/2015/10/storing-your-data-on-tvos.html). This is probably due to the fact that the Apple TVs capacity is either 32GB or 64GB, which is not that much when considering the Apple TV will be used for content-heavy and/or entertainment apps. Other relevant sizes for on-demand resources are defined on the [documentation](https://developer.apple.com/library/prerelease/tvos/documentation/FileManagement/Conceptual/On_Demand_Resources_Guide/PlatformSizesforOn-DemandResources.html):

| Item | Size |
|---|---|
| **tvOS App bundle** - The size of the [sliced](https://developer.apple.com/library/watchos/documentation/IDEs/Conceptual/AppDistributionGuide/AppThinning/AppThinning.html) app bundle downloaded to the device | 200MB |
| **Tag** - A string that identifies a set of resources| 512MB | 
| **Initial install tags** - The total sliced size of the tags marked for initial install | 2GB |
| **Initial install and prefetched tags** - The total sliced size of the tags marked for initial install and the tags marked for prefetching.| 4GB |
| **In use on-demand resources** - The total sliced size of the tags in use by the app at any one time. A tag is in use as long as at least one `NSBundleResourceRequest` object is accessing it. | 2GB |
| **Hosted on-demand resources** - The total pre-sliced size of the tags hosted on the App Store | 20GB |

As recommended on the [developer forums](https://forums.developer.apple.com/thread/18248):

*If you have content that is required for initial app launch, you can include up to 2GB of tags as initial install, which means the assets are included in the initial download from the App Store before the app can be launched. You can also mark assets for "prefetch", which means that the app can be launched even if they haven't been downloaded, but that tvOS will automatically start downloading them in the background when the app is purchased.*

Tags are used to identify a set of resources that need to be downloaded, note that resources could belong to multiple tags. Even though tags are limited to 512MB it is probably best to have much smaller tags otherwise your app will need to wait longer while all the pack is downloaded.

At any one time the app can use up to 2GB of static resources which is plenty even for the graphic intensive games. It is important to note that this extra data is cached by the OS, so the app can request when it needs resource and the OS will either download them or retrieve them from a cache. This means that depending on the disk space left on the Apple TV resources may be needed to be re-downloaded frequently or not. This is important to understand when designing your app around ODR since there is no guarantee that resources will be immediately available when requested if previously downloaded.

##Implementing on-demand resources

Apple has made available a guide to [downloading and accessing on-demand resources](https://developer.apple.com/library/prerelease/ios/documentation/FileManagement/Conceptual/On_Demand_Resources_Guide/Managing.html) that answers most questions. Luckily, it is quite straightforward to setup this and test on Xcode.

Firstly, on your xcassets set the resources tags the images belong to (images can belong to multiple tags):

<p align="center">
   <img src="/assets/images/xcassets.png"/>
</p>

On the project settings define which tags need to be "initial install tags", "prefetched tags" or "download on demand tags".

<p align="center">
   <img src="/assets/images/project_settings.png"/>
</p>

Note that it does not look like these states can be simulated as when running the app for the first time all resources appear as not downloaded regardless of the which category they belong to.

<p align="center">
   <img src="/assets/images/debugging_no_downloads.png"/>
</p>

To download the images from a tag 

```swift
let tagString = "RedTag"
// Create a resource request with the required tag(s)
let resourceRequest = NSBundleResourceRequest(tags: [tagString])

// Optionally define the loading priority if downloading multiple resources,
// use NSBundleResourceRequestLoadingPriorityUrgent if resources are required now
resourceRequest.loadingPriority = NSBundleResourceRequestLoadingPriorityUrgent

// Check if the resources are cached first
resourceRequest.conditionallyBeginAccessingResourcesWithCompletionHandler({ (resourcesAvailable : Bool) -> Void in
  if (resourcesAvailable) {
    // Do something with the resources
    print("Resources originally available")
  } else {
    // Resources not available to they will need to be downloaded
    // using beginAccessingResourcesWithCompletionHandler:
    print("Resources will be downloaded")
    resourceRequest.beginAccessingResourcesWithCompletionHandler({ (error : NSError?) -> Void in
      if (error != nil) {
        print("Failed to download resources with error: \(error)")
      } else {
        // Do something with the resources
        print("Resources downloaded successfully")
      }
    })
  }
})
```

Optionally set the preservation priority of a tag so that the operating system will know which resources to purge first.

```swift
let preservationPriorityForRedTag = 0.5
NSBundle.mainBundle().setPreservationPriority(preservationPriorityForRedTag, forTags: [redTagString])
```

Once the resources are available, they can be accessed as they normally would, for example:

```swift
let redImage = UIImage(named: "Red.png")
[redTagString])
```

Once the resources are no longer used Apple recommends to call `endAccessingResources()` on the request, *Call this method as soon as you have finished using the tags managed by this request. If needed, this method will be called by the system when the resource request object is deallocated.* 

The progress of resources requests can be observed using KVO on the `fractionCompleted` property of the `progress` of the resources request, this `progress` can also be used to manage the request state, for example:

```swift
resourceRequest.progress.pause()
resourceRequest.progress.resume()
resourceRequest.progress.cancel()
```

Resources can be purged from the simulator easily from Xcode:

<p align="center">
   <img src="/assets/images/debugging_red_blue_downloaded.png"/>
</p>

##Demo

The basic code to setup ODR explained on this article is on [GitHub](https://github.com/lucianomarisi/OnDemandResourcesExampleApp). Generally speaking, it may be more appropriate to wrap the methods of ODR in an object that is custom to your needs. This way your app can manage the different states of the tags as they are needed. This is important as the priority of the resources being used will vary throughout the lifecycle of any application and ideally it would be best if the OS does not purge resources that are currently being used.