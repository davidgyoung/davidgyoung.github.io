---
layout: post
title: Beacon Detection With Android 8
author: David G. Young
---
{% include JB/setup %}

One of the most important ways apps use Bluetooth beacons is to get a wake up when the beacon first comes into range.   If your app is running when this happens, this is no problem.  But what if you want your app to take action when a beacon appears, perhaps hours or days after the user last launched it?

This has always been tricky on Android, because of the lack of operating system support for both beacons and for background detection.  Before Android 8, tricky workarounds were necessary to make this happen.

**Android 8 includes changes that bring both good and bad news.  The bad news is that the existing workarounds will no longer work to let you detect beacons when you app is not in the foreground.  The good news is that new APIs give you new ways of doing these same detections**, and for the most important situations, detections will be just as fast while using less battery.

The table below summarizes the way background beacon detections work on various Android versions.  The top of the table describes the techniques supported by each Android version, and the bottom of the table shows how this converts to detection times.

---

|                                            | 4.3-4.4.x     | 5.0-7.x       | 8.0           |
| ------------------------------------------ |:-------------:|:-------------:|:-------------:|
| Allows long-running scanning services      | YES           | YES           | NO            |
| Supports JobScheduler scanning             | NO            | YES           | YES           |
| Supports bluetooth scan filter             | NO            | YES           | YES           |
| Sends bluetooth detections as intents      | NO            | NO            | YES           |
| Detection possible after reboot            | YES           | YES           | YES           |
| Detection possible after task manager kill | YES           | YES           | YES           |
| Typical secs to detect first beacon        | 150\*         | 5             | 5             |
| Typical secs to detect second beacon       | 150\*         | 150\*         | 450           |
| Typical secs to detect beacon disappear    | 150\*         | 150\*         | 450           |
| Typical secs to detect after kill          | 150\*         | 5             | 450           |
| Maximum secs to detect a beacon            | 300\*         | 300\*         | 1500          |


\* Android Beacon Library with the default backgroundScanPeriod of 300 seconds.  This may be adjusted to a higher or lower value, but lower detection times use proportionally more battery.

**Table 1. Technology Support and Beacon Detection Times Across Android Versions**

---

You'll notice from the table above that the typical background detection time remains pretty low at about 5 seconds on Android 8.  But after that initial background detection, finding a second beacon can be much slower if the first beacon remains in the vicinity.  And in the worst-case scenario, this can be up to 1500 seconds (25 minutes).  To understand why this is true, we have to understand what is changing in Android 8.

## What's Changing?

Apps on Android 4.3-7.x used long-running background services or alarms to periodically look for beacons in the background.  Android 8 implements new rules prohibiting long running background services in order to save battery.  Apps that use background scanning services to look for beacons won't work on Android 8.  The operating system will kill the background services about 15 minutes after the app was last in the foreground.  That means no more beacon detections after this 15 minutes when using background services to do the scanning.

Before Android 8, the Android Beacon Library would by default schedule a scan to happen every 300 seconds (5 minutes) in the background.  This was the primary means of finding beacons on 4.x, and used as a backup on 5.0-7.x for the cases where hardware scan filters were all used up, didn't work, or if beacons were already in the vicinity and the library just needed to check periodically for new ones.  This guaranteed a detection to happen in 300 seconds, but on average would happen in half that time -- 150 seconds.

Unfortunately, Android 8 no longer lets you do this.  You can't schedule background services to keep running constantly, looking for beacons every 5 minutes.  You can't use the AlarmManager to wake up your app every 5 minutes to do the same.

## The New Way: Fast Detections

Fortunately, Android 8 is providing a new tool for detecting beacons in the background.   New bluetooth scanning APIs let you specify a filter to match the BLE advertisement pattern you are looking for.   You can then start a scan, and request your app be woken up with an Intent whenever the pattern is matched.  Here's some code that sets that up:

```java
ScanSettings settings = (new ScanSettings.Builder().setScanMode(ScanSettings.SCAN_MODE_LOW_POWER)).build();
List<ScanFilter> filters = getScanFilters(); // Make a scan filter matching the beacons I care about
BluetoothManager bluetoothManager =
            (BluetoothManager) mContext.getApplicationContext().getSystemService(Context.BLUETOOTH_SERVICE);
BluetoothAdapter bluetoothAdapter = bluetoothManager.getAdapter();
Intent intent = new Intent(mContext, MyBroadcastReceiver.class);
intent.putExtra("o-scan", true);
PendingIntent pendingIntent = PendingIntent.getBroadcast(mContext, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
bluetoothAdapter.getBluetoothLeScanner().startScan(filters, settings, pendingIntent);
```

The above code will set an Intent to fire that will trigger a call to a class called `MyBroadcastReceiver` when a matching bluetooth device is detected.   You can then fetch the scan data like this:

```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
      int bleCallbackType = intent.getIntExtra(BluetoothLeScanner.EXTRA_CALLBACK_TYPE, -1);
      if (bleCallbackType != -1) {
        Log.d(TAG, "Passive background scan callback type: "+bleCallbackType);
        ArrayList<ScanResult> scanResults = intent.getParcelableArrayListExtra(
                                               BluetoothLeScanner.EXTRA_LIST_SCAN_RESULT);
        // Do something with your ScanResult list here.
        // These contain the data of your matching BLE advertising packets
      }
    }
}
```

Using the above, we can get a detection of the first new beacon being around when our app isn't running on Android 8 as we could when the app was passively sitting in the background on iOS 5-7.x waiting for a filtered scan to come in.

But after we see this first beacon, the background running time limits kick in.   The OS will kill the app within 15 minutes of this detection, so if we need to keep looking for a second beacon in the vicinity, and it doesn't show up for 16 minutes, our app can no longer be running.


## The New Way: Periodic Detections

Fortunately, Android does give us another tool here:  the JobScheduler.  This is a relatively new means of performing periodic activities (introduced in Android 5), even if your app is not running.  A scheduled job may be set up periodically to look for beacons.  This is important as a backup if fast detections described above don't work for some reason.   The most common case is when another beacon has already been discovered, remains in the vicinity, and you need to know when other beacons come into and out of view.

This is where the JobScheduler helps us.  We can set up a job to run periodically to look for the beacons around us and report them to our app.

But this mechanism comes with limitations.  Android 8 limits periodic jobs to being scheduled at most every 15 minutes (900 seconds) meaning it will may take this long to detect a new beacon or detect when a beacon disappears, although the law of averages says the mean detection time will usually be half that much (450 seconds).  You can try to schedule a job to run more than every 15 minutes, but the operating system will log something like this:

```
06-07 22:15:51.361 6455-6455/org.altbeacon.beaconreference W/JobInfo: Specified interval for 1 is +5m10s0ms. Clamped to +15m0s0ms
06-07 22:15:51.361 6455-6455/org.altbeacon.beaconreference W/JobInfo: Specified flex for 1 is 0. Clamped to +5m0s0ms
```

The maximum time, however, can actually be significantly more than 900 seconds.  This is because Android will sometimes defer periodic jobs to be even longer than the nominal minimum allowed interval of 900 seconds.  In theory this "flex" interval is supposed to abide by the parameter set in the API.  As you can see from the second line above, even if you request a flex of 0 seconds, the OS will refuse to use that and use the minimum flex, which appears to be 5 minutes in Android 6.  This means you job 15 minute job should have at most 20 minutes between runs.  But even this doesn't always happen.  In my tests, I saw cases where two jobs set up on a nominal 15 minute periodic schedule were executed 25 minutes apart.  In that case, it would have taken 1500 seconds to detect a new bacon that appeared right after the last job finished.  Check out the scan job run in the log excerpt below at 2:21 a.m.   Note that it happened 1530 seconds after the previous run.

```
06-07 22:25:51.380 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@7188bc6
06-07 22:41:01.227 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@382ed7b
06-07 22:55:51.373 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@203c928
06-07 23:10:59.083 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@dc96415
06-07 23:25:51.371 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@68bed2e
06-07 23:40:59.142 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@c295843
06-07 23:55:51.369 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@cd047e4
06-08 00:10:59.082 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@8009a61
06-08 00:25:51.368 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@f1fa2ca
06-08 00:40:59.085 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@88dddef
06-08 00:55:51.374 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@eb2b360
06-08 01:10:51.670 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@9bca225
06-08 01:25:51.383 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@871c8fe
06-08 01:45:51.404 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@3bf42d3
06-08 01:56:12.354 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@c3d4e34
06-08 02:21:51.771 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@1557571
06-08 02:37:01.861 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@e2c879a
06-08 02:52:11.943 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@c9f0d7f
06-08 03:07:22.041 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@4e0cab0
06-08 03:23:12.696 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@1139a7d
06-08 03:38:22.776 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@e06b8f6
06-08 03:52:12.792 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@74147eb
06-08 04:08:32.872 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@90d9fec
06-08 04:21:12.856 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@a4abd49
06-08 04:38:42.959 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@741d912
06-08 04:50:12.923 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@15bfe17
06-08 05:08:53.047 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@fa229e8
06-08 05:19:13.050 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@b0e49d5
06-08 05:39:03.142 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@18823ee
06-08 05:54:13.212 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@a72fc03
06-08 06:10:51.850 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@3fb84a4
06-08 06:26:01.917 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@53d6c21
06-08 06:41:11.994 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@848958a
06-08 06:56:22.053 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@43cdaf
06-08 07:06:32.119 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@5318c20
06-08 07:29:12.356 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@34f102d
06-08 07:44:22.431 6455-6455/org.altbeacon.beaconreference I/ScanJob: Running periodic scan job: instance is org.altbeacon.beacon.service.ScanJob@4d2e9e6
```

These numbers are based on a limited sample size in an overnight test of about 9 hours, so it is certainly possible that even longer periods between scans may happen on Android 8.  We know it can be up to 25 minutes between scans.  But it could be even more.

The fact that such delays might sometimes happen with Android 8 is disappointing.  But at least we understand why.  On iOS, we also see similar detection delays  in some cases, but the closed-source  code means we have no idea why.

When testing, just remember that this 15 minute interval is typical, but not exact. Expect scans in these cases to happen every 10-25 minutes.

## Upgrading Your App

If you have a beacon app targeting earlier versions of Android that relies on long-running services for detections (e.g. the Android Beacon Library), you'll need to modify these apps and submit them to the store so they'll continue to be able to detect beacons in the background on Android 8.  If you don't, people who run your unmodified app on Android 8  will only be able to detect beacons for 15 minutes after last being in the foreground.

For users of the Android Beacon Library, the upgrade process is simple.  Simply upgrade to the latest Android Beacon Library 2.12+.  This version uses all of the techniques above to deliver the best detection times possible as described in the table above, in a way that is background compatible.  The old means of scanning and detection times will still apply for apps running on Android 4.x-7.x.  Only apps deployed on Android 8 will use the new techniques.  So the slower worst-case detection times won't affect users on earlier operating system versions.

If you are using this library and have customized the background scan intervals to be more frequent than 15 minutes, be aware that the scans won't happen. at that frequency, and even scans set to happen every 15 minutes won't happen at precisely that time interval.   So be aware.  When your boss asks you why your app isn't triggering right away, you can answer with everything you read above.

