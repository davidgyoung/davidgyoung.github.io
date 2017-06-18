---
layout: post
title: Extending Background Ranging on iOS
author: David G. Young and Scott Yoder
---

One of the most common use cases for beacon-enabled applications is to perform an action when a user gets close to a specific location.  Since beacons typically have a transmitting range of up to 50 meters, it’s often not appropriate to trigger that far away.  A gas station app, for example, shouldn’t prompt a user to pay at the pump when the user’s phone detects the beacon from across the street.

There are a couple of strategies to solve this.  One involves turning down the transmitter power of the beacon so it can’t be detected until the user is only a few meters away.  The RadBeacon models sold by Radius Networks allow adjusting the transmitter power for this exact reason.   But when very close triggering is desired, this approach can sometimes lead to actions not being triggered at all.  Received signal levels can vary depending on mobile device model, and where the device is placed.

A second approach involves tracking the beacon in the background, noting its estimated distance, and only triggering an action when the beacon is estimated to be within a specific range.  This approach is problematic on iOS, because CoreLocation generally allows only 10 seconds of ranging time when an app is in the background.  If a beacon is first detected at 50 meters, and a person is approaching the beacon at one meter per second, the mobile device will still be 40 meters away when iOS suspends the app and stops it from ranging.

The good news is that it is possible to extend background ranging time on iOS.  If your app is a navigation app, you can specify location updates in the “Required background modes” in your Info.plist.  But this approach makes it harder to get AppStore approval -- you have to convince reviewers that your app is providing navigation services to the user.  This probably isn’t true for many apps that simply want to use beacons to trigger at a specific distance.

Fortunately, you can still extend background ranging time without requesting special background modes.  The time you can get is limited -- only three minutes.  But this clock restarts each time your app is woken up in the background, meaning you can get an extra three minutes of ranging time each time your app detects a beacon (enters a beacon region) or stops seeing beacons (exits a beacon region.)

<i>[Continue reading this blog post](http://developer.radiusnetworks.com/2014/11/13/extending-background-ranging-on-ios) on the Radius Networks website</i>
