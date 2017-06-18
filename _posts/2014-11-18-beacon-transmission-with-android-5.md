---
layout: post
title: Beacon Transmission With Android 5.0
author: David G. Young
---

One of the most exciting new features of Android 5.0 is the support for Bluetooth LE peripheral mode, which makes it possible to turn an Android device to a beacon transmitter.  Radius Networks is proud to release the new QuickBeacon app for Android 5.0, which creates a configurable beacon supporting both the AltBeacon and Apple iBeacon&#8482;   formats.  This app is available for free for a limited time only in the [Google Play Store](https://play.google.com/store/apps/details?id=com.radiusnetworks.quickbeacon), and works with the new Nexus 6 phone and Nexus 9 tablet.

QuickBeacon for Android makes for an extremely versatile software beacon transmitter, because unlike iOS-based transmitters, it allows you to keep transmitting even when the app is no longer in the foreground.  This makes it suitable for production use on tablets used as point-of-sale (POS) devices where the transmitter app will not always be in the foreground.

The app also allows you to configure the transmitter power, allowing you to adjust how far away the beacon transmission will be detected.  Similarly, you can adjust the transmission frequency of the beacon, allowing you to save battery power if frequent transmissions are not needed.  Neither of these adjustments are available on iOS or OS X-based transmitters.

It is important to note that not all devices with Android 5.0 can send beacon transmissions.  The Android device must have Bluetooth LE hardware and it must have peripheral mode enabled by the device manufacturer.   Google surprised many people when it [disabled peripheral mode for their Nexus 5 and Nexus 7 devices](https://code.google.com/p/android-developer-preview/issues/detail?id=1570), meaning these devices cannot transmit beacon advertisements using QuickBeacon or other software.  This can be confusing for folks using the Google Play store, because it does not have a capabilities filter for BLE peripheral mode.  As a result, QuickBeacon must be installed and tested on each Android 5.0 model before it is known whether or not the app is compatible with that model.  If the Android model does not support BLE peripheral mode, the app will tell you right away.

<i>[Continue reading this blog post](http://developer.radiusnetworks.com/2014/11/18/beacon-transmission-with-android-5) on the Radius Networks website</i>
