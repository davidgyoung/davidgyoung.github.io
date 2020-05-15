---
layout: post
category :
tagline: ""
tags : []
title: Android iBeacon Library Now Availalble
---
{% include JB/setup %}

Note:  This blog post was originally published on the Radius Networks website, but was taken down in July of 2014 after representatives of Apple requested changes to infromation published under terms of the iBeacon license agreement.

<hr/>

The release of iOS 7 last month has made it possible to develop some pretty cool proximity-aware apps for iPhone and iPad devices. But what about Android? With our new Android iBeacon Library, you can now build the same cool proximity-aware apps on the world's largest mobile operating system. We've released it under the Apache 2 license, which means you can embed it in your own applications and sell it on the Google Play store.

## How it Works
An iBeacon works by advertising Bluetooth Low Energy packets once per second. It sends out a three part unique identifier that can be received by your app in real time. When your app sees the identifier, it has a pretty good idea where it is. iBeacons have a range of only a few hundred feet, and you can get an estimate of the actual distance to the iBeacon by analyzing the signal strength. In iOS 7, built-in APIs perform all these functions for you. On Android, you can get very similar APIs by using our library. On both platforms, the APIs perform two basic functions:

* Monitoring - The Monitoring API allows your app to look for one or more iBeacons and get notified when they come into range or go out of range.
* Ranging - The Ranging API allows your app to get updates once per second on the estimated distance to the iBeacons that it sees.

## Device Support

This library is possible because the July release of Android 4.3 brought OS-level support for Bluetooth LE. Not many Android devices have received that update so far (although my Nexus 4 has), but carriers are expected to roll it out to many higher-end phones like the Galaxy S3/S4 in coming months. In addition to Android 4.3 (SDK version 18), devices also have to have a low energy Bluetooth chipset, sometimes called BLE or Bluetooth 4.0. As of September 2013, Android devices known to have BLE include: Samsung Galaxy S3/S4, Samsung Galaxy Note II, HTC One, Nexus 7 2013 edition, Nexus 4, HTC Butterfly, Droid DNA.

You can [read the full blog post in its original form](http://web.archive.org/web/20140124150238/http://developer.radiusnetworks.com/2013/10/07/android-ibeacon-library-now-available.html) through the Internet Wayback Machine.



