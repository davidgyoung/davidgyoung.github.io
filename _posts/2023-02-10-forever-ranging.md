---
title: Forever Ranging
---

Apple has always placed restrictions on how long iPhone apps can range for Bluetooth beacons in the background.  While apps can range for unlimited periods of time as long as they are in the foreground (visible on teh screen), once they go to the background, ranging updates generally stop after 10 seconds.  They may resume again for another 10 seconds on certain events like a beacon region entry or exit, but it won't last long.

## Make It Last

The good news is that it is possible to keep ranging going forever.  Setting it up is a bit tricky, and will use significant battery if you are around beacons for long periods of time.   But is legal to do for App Store distribution provided that your app makes it clear that it uses location in the background for an approved purpose.  

Here's what you need to do:

### App Setup:

 1. Put the following declaration in your Info.plist 
```
    <key>UIBackgroundModes</key>
    <array>
        <string>location</string>
    </array>
```
 2. Write code to obtain `NSLocationAlways` permission from the user.  

### Starting Ranging

When it's time to start ranging, you'll need to perform some additional steps.

 1. Start beacon ranging using the normal APIs.
 2. Start location updates at the lowest power setting (basically just using cell radio) with:
```
        locationManager.pausesLocationUpdatesAutomatically = false
        locationManager.desiredAccuracy = kCLLocationAccuracyThreeKilometers
        locationManager.distanceFilter = 3000.0
        
        if #available(iOS 9.0, *) {
          locationManager.allowsBackgroundLocationUpdates = true
        } else {
          // not needed on earlier versions
        }
        // start updating location at beginning just to give us unlimited background running time
        locationManager.startUpdatingLocation()
``` 

 3. Start a background task:
```
      var backgroundTask: UIBackgroundTaskIdentifier = UIBackgroundTaskIdentifier.invalid

      NSLog("Attempting to extend background running time")
      
      self.backgroundTask = UIApplication.shared.beginBackgroundTask(withName: "DummyTask", expirationHandler: {
        NSLog("Background task expired by iOS.")
        UIApplication.shared.endBackgroundTask(self.backgroundTask)
      })

    
      var lastLogTime = 0.0
      DispatchQueue.global().async {
        let startedTime = Int(Date().timeIntervalSince1970) % 10000000
        NSLog("*** STARTED BACKGROUND THREAD")
        while(true) {
            DispatchQueue.main.async {
                let now = Date().timeIntervalSince1970
                let backgroundTimeRemaining = UIApplication.shared.backgroundTimeRemaining
                if abs(now - lastLogTime) >= 2.0 {
                    lastLogTime = now
                    if backgroundTimeRemaining < 10.0 {
                      NSLog("About to suspend based on background thread running out.")
                    }
                    if (backgroundTimeRemaining < 200000.0) {
                     NSLog("Thread \(startedTime) background time remaining: \(backgroundTimeRemaining)")
                    }
                    else {
                      //NSLog("Thread \(startedTime) background time remaining: INFINITE")
                    }
                }
            }
            sleep(1)
        }
        NSLog("*** EXITING BACKGROUND THREAD")
      }
```

The above will keep giving you ranging updates forever, even in the background, until your phone reboots, the user kills your app, or the operating system kills your app due to low memory.

The data from the location update is not needed, but turning on the location update is a signal to the operating system (combined with the location background mode we set up in the Info.plist) that it should let you app keep running in the background indefinitely.  The background task lets iOS know that it is processing this data and therefore needs the running time.

## Battery Impact

Whenever you have beacon ranging enabled on iOS, the Bluetooth chip is doing a scan at the highest duty cycle.  The radio receiver is on 100% of the time.  While modern iPhones and their batteries are pretty tolerant to Bluetooth scanning for limited periods of time, if you leave this on all day, the user will notice the battery draining about twice as quickly as normal.

So be careful with this -- don't just turn on ranging forever without letting the user know (and getting user consent) to do something that will use a lot of battery.

The background location update we requrested, on the ohter hand, uses very little battery.  This is because the desired accuracy we requested, `kCLLocationAccuracyThreeKilometers`, only uses the cell radio for location.  Each time the cell tower changes, the app gets a location update from that cell tower.  Since the cell radio is on anyway, this causes no additional battery drain.

## Safe for the App Store

All of the above are legal techniques appropriate for apps to be published in the AppStore.  However, becuase you are requesting a location backround mode in Info.plist, your app must have some obvious user-facing benefit for using location in the backround.  If you have an app that navigates you around an obsticle course or tracks your umbrella to keep you from leving it behind, this will probably be okay.  

But if your app is a video game that passively looks for beacons to send location-targeted advertising to the user, that will probably be rejected.

