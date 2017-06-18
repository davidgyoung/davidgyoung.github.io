---
layout: post
title: iOS 7.1 Background Beacon Detection Times
author: David G. Young
---

The release of iOS 7.1 has led to some hope that the new release will improve beacon background detection times.  Repeating [earlier tests](/2013/11/13/ibeacon-monitoring-in-the-background-and-foreground.html) on iOS 7.0 shows this not to be the case — it can still take up to 15 minutes to detect an beacon in the background.

These tests used a iPhone 4S recently upgraded to iOS 7.1 running an [Beacon BackgroundDemo program available on GitHub](https://github.com/RadiusNetworks/ibeacon-background-demo).  The same program had been used to test background detection times on iOS 7.0, and the full test setup and procedure is described in an [accompanying blog post](/2013/11/13/ibeacon-monitoring-in-the-background-and-foreground.html).   For an beacon transmitter, the test used a [Radius Networks Dual Beacon Development Kit](http://store.radiusnetworks.com/collections/all) (with two transmitters), that send out advertisements at a frequency of 10Hz when enabled.   Cycling these transmitters on and off produced the following annotated log file (annotations start with #):

<i>[Continue reading this blog post](http://developer.radiusnetworks.com/2014/03/12/ios7-1-background-detection-times) on the Radius Networks website</i>