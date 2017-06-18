---
author: David G. Young
layout: post
title: CoreBluetooth Doesn't Let You See Beacons
---


So, is there any way to to monitor or range for any beacon regardless of its ProximityUUID on iOS?  The short answer is no.

The long answer involves trying a couple of different ways:

### CoreLocation APIs

Using the CoreLocation APIs, the obvious way to look for all beacons is to pass a nil value for the ProximityUUID when constructing a CLBeaconRegion.  But that doesn't work.  If you try this:

```objective-c
CLBeaconRegion *region = [[CLBeaconRegion alloc] initWithProximityUUID:nil identifier:@"myUniqueIdentifer"];
```

<i>[Continue reading this blog post](http://developer.radiusnetworks.com/2013/10/21/corebluetooth-doesnt-let-you-see-ibeacons) on the Radius Networks website</i>



