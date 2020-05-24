---
layout: post
title: Storing your data on tvOS, part 1
date: '2015-10-24T22:47:00.000+01:00'
author: nahuel_marisi
tags:
- tvOS
permalink: /2015/10/storing-your-data-on-tvos.html
---

Continuing with our tvOS series ([post 1](/2015/10/5-things-to-bare-in-mind-if-you-are.html), [post 2](/2015/10/interacting-with-new-apple-tv-remote.html)) we shall look at data storage options on tvOS in this two part article. This article is not specific to tvOS, however the limitations described here are related to the Apple TV and may be different on iOS or Mac OS X devices.

There are three main ways of storing data on tvOS:

1. The first one is the well known NSUSerDefaults. While the official documentation is rather silent on this subject, it seems that this method's limit on tvOS is 500 KB according to [Apple](https://forums.developer.apple.com/message/50696#50696).

2. The second method is iCloud Key Value Storage (KVS). This works in a similar fashion to NSUserDefaults but it's stored in the cloud instead. Furthermore, there is space for up to 1 Mb and data can be shared between tvOS and iOS devices if desired. So if you need more than 500 KB and / or you want to share data among devices, this is a method worth exploring.

3. The final method discussed in this article, is CloudKit. CloudKit allows you to store as much as you want, the only limitation being the user's iCloud storage limit. We shall discuss CloudKit in detail in the second part of this article. If you need to store relatively large amounts of data, this is clearly the preferred method. 

## NSUserDefaults

NSUserDefaults on tvOS works in the same way it does on Mac OS X or iOS. You must obtain the NSUserDefaults singleton, and then you can save or retrieve key-value pairs.

## Saving and retrieving values 

After you've obtain the NSUserDefaults singleton you can call setString, setBool, setObject, etc to store key-value pairs. To retrieve values you can call stringForKey, boolForKey, objectForKey, etc.
For example:

Saving:

```swift
NSUserDefaults.standardUserDefaults().setObject(textField.text,
            forKey: textKey)
NSUserDefaults.resetStandardUserDefaults()/
```

Retrieving:

```swift
guard let loadedText = NSUserDefaults.standardUserDefaults().objectForKey(textKey)
 as? String else {
            return
} 	
```

## iCloud KVS

You can think of iCloud KVS as a sort of NSUserDefauts in the cloud. On tvOS you get twice as much storage space (up to 1MB) and the ability to sync data among devices running iOS or tvOS.

### Enabling iCloud KVS in your app

Go to Xcode and enable the `com.apple.developer.ubiquity-kvstore-identifier` entitlement for your app:

<p align="center">
   <img src="/assets/images/enabling_icloud_kvo.png"/>
</p>

### Saving and retrieving values

In a similar fashion to NSUserDefaults, iCloud KVS uses the singleton `NSUbiquitousKeyValueStore` to store and retrieve values based on keys.

To save a value into the key-store you can get the shared delegate and then call the appropriate method, setFor... String, Bool, Object, etc. 

For example:

```swift
let store = NSUbiquitousKeyValueStore.defaultStore()
store.setString(textField.text, forKey: textKey)       
```


To retrieve a value, you use the same singleton, but call setFor.. String, Bool, Object, etc methods. For example:

```swift
let store = NSUbiquitousKeyValueStore.defaultStore()
textField.text = store.stringForKey(textKey)
```
 
 
### Keeping track of value changes

When using iCloud KVS, any device that is running your app can change the stored values. Furthermore, if you're using the same app identifier between your iOS app and your tvOS app, they can share data through iCloud KVS.

If you exceed the maximum iCloud KVS capacity, the notification will be issued with a value of `NSUbiquitousKeyValueStoreQuotaViolationChange`.

It's therefore convenient to register for the `NSUbiquitousKeyValueStoreDidChangeExternallyNotification` notification as soon as possible. In this way, you make sure your app starts off with the latest available data. For example, in the demo project, I've registered in the AppDelegate:

```swift
func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
    
    NSNotificationCenter.defaultCenter().addObserver(
        self,
        selector: "storeDidChange:",
        name: NSUbiquitousKeyValueStoreDidChangeExternallyNotification,
        object: NSUbiquitousKeyValueStore.defaultStore())
    
    // Download any changes since the last time this instance of the app was launched
    NSUbiquitousKeyValueStore.defaultStore().synchronize()
    
    return true
}   
```

In the following example, we print the keys that have changed on iCloud's server:

```swift
    
func storeDidChange(notification: NSNotification) {
    guard let userInfo = notification.userInfo else {
        return
    }
   
    guard let reasonForChange = userInfo[NSUbiquitousKeyValueStoreChangeReasonKey] as? NSNumber else {
        return
    }
    
    // Update local values only if we find server changes
    let reason = reasonForChange.integerValue
   
    if reason == NSUbiquitousKeyValueStoreServerChange ||
        reason == NSUbiquitousKeyValueStoreInitialSyncChange {
            
            // Get Changes
            let changedKeys = userInfo[NSUbiquitousKeyValueStoreChangedKeysKey] as! [AnyObject]
            let store = NSUbiquitousKeyValueStore.defaultStore()
            
            // Print keys that have changed
            for key in changedKeys {
                let value = store.objectForKey(key as! String)
                print("key: \(key), value: \(value)")
                
            }
    }
 }

```


### Resolving conflicts
When a device attempts to write a value to iCloud's KVS storage, iCloud will check to see if any recent changes have been made by other devices before storing your value. If changes have been made recently, iCloud will issue a `NSUbiquitousKeyValueStoreDidChangeExternallyNotification` forcing your app to update itself with the server's data. You must then decide how the server values should be dealt with.
For example, if your app is a game and tried to update the user's points to say 1000 but received a notification with the server data being 1500, you will assume that the server data is more recent and that your app should therefore take 1500 as the most up-to-date value instead of forcing the server to store 1000. 


## Demo
Everything discussed in this article can be found in a demo project in github here:
[Github](https://github.com/nmarisi/appletv-storage-demo)

