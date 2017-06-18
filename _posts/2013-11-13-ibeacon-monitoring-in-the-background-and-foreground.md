---
layout: post
title: Beacon Monitoring in the Background and Foreground
author: David G. Young
---

There has been lots of confusion about how Beacon monitoring and ranging work in the background on iOS.  In general ranging only works in the foreground, but monitoring updates can happen in the background.  There are two ways to configure background monitoring, and even when done properly updates can take a long time to come.  Because these delays, some people mistakenly believe that monitoring in the background doesn't work.

##Summary

The tables below summarizes whether you can get monitoring updates under various conditions, and how long it might take to get them.

When the app is in the foreground:

<style type="text/css">
  table.rsum {
    border-collapse: collapse;
    border: 1px solid black;
  }
  table.rsum td{
    border: 1px solid black;
    padding: 5px;
  }
  table.rsum th{
    border: 1px solid black;
    padding: 5px;
  }

</style>

<table class="rsum">
<tr><th>Condition</th><th>Max time to detect a region change</th></tr>
<tr><td>App ranging    </td><td>1 second</td></tr>
<tr><td>App not ranging</td><td>up to 15 minutes</td></tr>
</table>

When the app is not in the foreground:

<table class="rsum">
<tr><th>Condition</th><th>Max time to detect a region change</th></tr>
<tr><td>Phone awakened, notifyEntryStateOnDisplay=YES</td><td>1 second </td></tr>
<tr><td>Phone awakened, notifyEntryStateOnDisplay=NO</td><td>NEVER </td></tr>
<tr><td>UIBackgroundModes=location ON</td><td>up to 15 minutes </td></tr>
<tr><td>UIBackgroundModes=location OFF</td><td>up to 15 minutes </td></tr>
</table>

Note: "Phone awakened" means pressing either the shoulder button or the home button when the phone display was off.  The detection times, in this case, reflect the time from when the screen turns on.


## How to maximize beacon responsiveness

The above tables show that two things are very important to maximizing the frequency of beacon monitoring updates:

<i>[Continue reading this blog post](http://developer.radiusnetworks.com/2013/11/13/ibeacon-monitoring-in-the-background-and-foreground) on the Radius Networks website</i>
