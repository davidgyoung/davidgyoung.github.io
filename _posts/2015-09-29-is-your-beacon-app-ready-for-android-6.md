---
layout: post
title: Is Your Beacon App Ready for Android 6.0?
author: David G. Young
---

The new Android 6.0 release has several important changes that affect apps detecting bluetooth beacons.  If you have a beacon-based app already in the Play Store, or are planning a new beacon-based app, you'll need to make updates to keep your app from breaking as users transition to Android 6.0.

While this article focusses largely on the impact for users of the <a href='https://altbeacon.github.io/android-beacon-library/'>Android Beacon Library</a> and Radius Networks' <a href='http://proximitykit.radiusnetworks.com'>ProximityKit</a> and  <a href='http://campaignkit.radiusnetworks.com'>CampaignKit</a> libraries that are built upon it, the same issues described here apply to any app that detects beacons.  If you are using a different vendor's SDK, it is important to make sure they will continue to work on Android 6.0.

## Runtime Permissions

The biggest change for beacon apps in Android 6.0, codenamed Marshmallow, and sometimes called just "M", has to do with permissions.  Just like iOS, Android now implements permissions at runtime instead of the traditional way of granting permissions at install time.  Apps designed for Marshmallow (SDK 23 and above) must add code to prompt users for some permissions after the app starts up, otherwise they will not be granted.

Not all permissions, however, work this way.  Permissions marked as `PERMISSION_NORMAL` are still granted the old fashioned way: at install time.  For beacon apps, two important permissions continue to follow the old model:  `android.permission.BLUETOOTH` and `android.permission.BLUETOOTH_ADMIN`, both of which is needed to scan for beacons.  Because these permissions still use the old behavior, nothing really changes with them in Marshmallow.

<i>[Continue reading this blog post](http://developer.radiusnetworks.com/2015/09/29/is-your-beacon-app-ready-for-android-6) on the Radius Networks website</i>
