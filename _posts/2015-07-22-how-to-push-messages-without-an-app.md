---
layout: post
title: How To Push Beacon Messages Without an App
author: David G. Young
---

One of most common things businesses want to do with bluetooth beacons is to solicit customers to get their mobile app.  This has long been impossible, because beacons can't be detected without an app already on the phone to look for them.  For folks wanting to get visitors to use their app, it's a classic chicken and egg problem.

As of today, Google is taking steps to change this.  The company is updating its Chrome for iOS web browser to automatically integrate with its Physical Web project, which detects [beacons transmitting the Eddystone-URL format](http://store.radiusnetworks.com/products/eddystone-devkit).  When Chrome for iOS detects one of these beacons, it shows information about the website to the user, allowing the user the option of visiting it within the browser.

### How it Works

<img src="/img/chrome-configure.png" width="214" align="right" />
The new Chrome feature works within the Chrome Today widget on the Notification Center Today view.  For those unfamiliar with the feature, the Today view is what you see when you swipe down from the top of your iPhone. By default it shows your calendar and  stock information, but you can add other App widgets by tapping the Edit button at the bottom of the screen.  If you have Chrome for iOS installed, the Chrome Today widget will show up in the list of the ones you can add.

If the Chrome Today widget is activated, the first time a Eddystone-URL beacon is discovered, the website's title and description will appear in the Chrome Today widget.  Tapping on the website launches Chrome to the web page address transmitted by the beacon.  This web page can be optimized for mobile, serving a welcome message to the visitor that can solicit a download of a mobile app to further take advantage of beacons.

<i>[Continue reading this blog post](http://developer.radiusnetworks.com/2015/07/22/how-to-push-messages-without-an-app) on the Radius Networks website</i>
