---
title: Making Magic Without Permission 
---

Some of the coolest things you can do with Bluetooth beacons involve making magic happen when you become near or far from something.  You can unlock a car, get alerted about leaving your umbrella behind, open the door to a parking garage or sync data with your dash cam.  What makes these things seem magically cool is they happen automatically when you come within range or go out of range -- no need to open an app or tap a button.

## The Problem

But an increased awareness of privacy has caused Apple and Google to crack down on how apps access location data, requiring explicit permission from users  to access location sensors.   And making your app wake up  and do something when it goes in or out of range traditionally requires getting "all the time" location permission from the user to detect a Bluetooth beacon or a geofence.

That's bad news.  Because users are increasingly unwilling to grant apps location permission, especially "all the time".  This is typically because they don't trust that the app won't misuse the permission to spy on them and send data bout their every movement to advertisers (or worse.)  A number of users left one star ratings and scathing reviews for one of my apps because it asked for location permission "all the time" so it could detect a parking garage gate and open it automatically without needing to open the app.  No amount of assurance about what the app does would assuage these users. 

Product managers and marketing departments are increasingly taking note.  New mobile projects increasingly have a requirement that location permission is not required -- or at least that the "all the time" variant of location permission  not  be required.  What is an engineer to do?  How can you make the magic happen -- build an app that automatically does something without a manual gesture -- without location permission?

## The Alternatives

An obvious alternative is to require the user to launch the app and gesture to a button to launch the action.  That might be fine if it were 2006 -- today we can do better.

Finding better alternatives means understanding which proximity sensors require location permission and what sensors do not.  Anything that causes the operating system to reveal your latitude and longitude (GPS, WiFi or Bluetooth through location services) requires location permission.  Similarly, scanning for Bluetooth beacons (iBeacon through CoreLocation on iOS or any common Bluetooth beacon format  on Android) requires location permission.  

If you only care about avoiding location "all the time" permission, then iOS does give you some ability to use the above if your app is manually launched, and location tracking is started before the app is put to the background.  In this case, iOS only requires "while using" location permission, but lets your app keep accessing the sensors in the background.  A blue location icon is shown to the user to alert them the app is continuing to use location in the background.  This allows the magic to happen if the user launches the app after phone boot, and the app keeps running in the background.  Is this really magic?  If so it is pretty lame magic.

But full magic is still possible with the last alternative:  using non-beacon Bluetooth detections.  Both iOS and Android allow apps to be launched into the background and perform automatic actions  based on detecting Bluetooth devices that are NOT beacons.  The details differ between Android and iOS, but the same principles apply.  The technology that allows this is what is behind  your phone's ability to automatically connect to your smartwatch or your fitness tracker.  But this technology can be used for nearly any purpose for which you would use Bluetooth beacons, and you can do so without asking the user for location permission at all.  You only need to ask the user for Bluetooth permission -- something that is much easier to get them to do.

## How it Works: IOS

On iOS, the rules are pretty simple.  You cannot use iBeacon (accessed with the CoreLocation framework) without obtaining location permission from the user.  But you can use Eddystone beacons or any other Bluetooth advertisement using a GATT Service advertisement format (accessed with the CoreBluetooth framework)  by merely requesting Bluetooth permission from the user.  

Getting access to do this in the background is simply a matter of adding the Bluetooth Central key to your required background modes in your app's Info.plist.  This will let your app run in the background in response to a Bluetooth event like detecting a Bluetooth advertisement, connecting to or disconnecting from a device, or exchanging data with a device.  If you need to automatically launch your app into the background on one of these Bluetooth events, you need to implement what's called Bluetooth State Restoration.  This is the trickiest part of the mix, requiring changes to the AppDelegate to execute special code on app launch, special initialization of CoreBluetooth and implementation of a special CBCentralDelegate method.  Getting this is just right can be tricky, but if you do it right, you will be rewarded with magic app behavior without obtaining the dreaded location permission.

## How it Works: Android

On Android, the rules are a bit more restrictive.  Unlike iOS, Android APIs never discerned between iBeacon and other Bluetooth beacon formats. Android's designers knew you could use Bluetooth devices for discerning location, so they originally required both BLUETOOTH_ADMIN and LOCATION permission to scan for any Bluetooth device.  But these are not runtime permissions, meaning they are only shown to the user at install time, something that almost everybody ignores.  When it comes to runtime permissions, apps need request only BLUETOOTH_SCAN from the user with the "neverForLocation" userPermissionFlags.    This will present a a basic Bluetooth permission dialog to the user without any need to request location permission.  The only drawback is that Android will filter out any devices from scan results that match the iBeacon, AltBeacon or Eddystone formats.  This means that you must use proximity devices that advertise a different kind of advertisement, typically a GATT Service advertisement like mentioned in the iOS section (although on Android a non-iBeacon non-AltBeacon manufacturer advertisement will work, too.)

While the format rules on Android are complex, the rest of the rules are effectively the same as for using iBeacon, AltBeacon or Eddystone with location permission (you just can't use those specific formats without location permission).  You can detect in the background and auto-launch your app using a number of techniques, the most useful being Bluetooth scans backed by Android Intents, which always auto-launch your app on detection.

You can even use my Android Beacon Library to do all these things without location permission -- you just need to follow the rules above by using a different beacon format and modify the permissions int he AndroidManifest.xml to have the  "neverForLocation" userPermissionFlags set on BLUETOOTH_SCAN.  Everything else can work the same.

## Cross Platform Considerations

If you want your system to work on both iOS and Android, it is best to pick a custom non-Beacon Bluetooth advertisement format that works on both platforms without location permission.   That means no iBeacon, AltBeacon or Eddystone, because iBeacon requires location permission on both platforms and AltBeacon and Eddystone require it on Android.   If you want background detection, that means no manufacturer advertisements (iOS does not allow scanning them in the background regardless of location permission).  This leaves GATT Service Advertisements that can be detected on both Android and iOS in the background without location permission.  

You won't be able to transmit GATT Service Advertisements on iOS with any additional identifiers (the platform disallows advertising GATT Service Data) so only Android can be a transmitter.

An example advertisement format using a GATT Service Advertisement with a 128-bit UUID, and 7 bytes of attached service data (for a three part identifier) is shown in the examples.  Because of the limited space of bluetooth advertising packets, there isn't room for transmitters to include the usual two AD Types of advertisement pakcets (Full list of 128-bit Service UUIDs and 129-Bit Service Service Data).  There is just room for the second AD type.  But this is sufficient for both iOS and Android to discover the advetisement in the background.  Here's an example "not a beacon" advetisement layout using the Android Beacon Library's Beacon Parser layout expression: "s:0-15=4c052726-cd97-4dde-9356-212cc1327a84,m:16-16=00,i:17-18,i:19-20,i:21-22,p:-:-59".  This defines a sample 128-bit Service UUID and 7 bytes of service data starting with a 00 to idenitify the packet type, and three two byte identifiers used to disctinguish one transmitter from another.

## The Downsides

There are some downsides to this non-beacon Bluetooth technique.  On iOS, if the user kills the app by swiping it off the screen, a Bluetooth detection won't re-launch the app into the background for processing (unlike the iBeacon technique which will -- iBeacon is particularly powerful for its ability to re-launch apps after the user kills it.) If your app really must have the ability to detect after the user kills the app, and can't require location "all the time" permission as needed for iBeacon, get ready to argue with your product manager -- this is a case where you can't have your cake and eat it too.

On Android, the big downside comes from the fact that  you can't use any standard beacon formats.  This means you cannot buy off the shelf beacons for your project.  You will need a custom Bluetooth advertisement format.   (You can use one like described above.)  If you already have.a custom hardware device on your project, this is not problem.  Just modify its advertising to meet these rules.  But if you do not have a custom device, and you are not relying on an Android transmitter (or a Linux/Windows computer) as the source of the Bluetooth advertisement, then you will likely need to get a Bluetooth device you can customize to send a special advertisement.   If you have a developer with the firmware development skills, this is not hard to do -- but it does mean that you will need special tooling and Bluetooth hardware with debug connectors that allow you to modify the firmware.  If you expect  thousands of Bluetooth devices in the field, the custom firmware will need to be flashed by the hardware manufacturer.

## Is This Really Legal?

Yes.  Whether it makes sense or not, the designers of iOS and Android have made an artificial distinction between Bluetooth beacons that require location permission (iBeacon only on iOS and also AltBeacon and Eddystone on Android) and other Bluetooth devices that do not.  This has always been a bit silly, because there really is not a clear line between non-beacon Bluetooth devices that advertise (speakers, security cameras, health sensors, watches) and Bluetooth beacons.  Any Bluetooth device in a fixed location that transmits a static identifier (and most non-Beacon devices do advrertise a static MAC address, at least in pairing mode) can be used to infer the physical location of that user, much in the same way as WiFi SSIDs are used to infer location by the Google street view vehicles who scan every neighborhood on the planet.   

What really makes a Bluetooth device a beacon anyway?  What is the point of requiring location permission for some Bluetooth devices but not others?

Like many regulations, it doesn't really make sense if you look at it too closely.  Prohibiting scanning common beacon types without location permission might make it a little harder for nefarious app developers to secretly snoop on app users' location without permission.  But if you are willing to create your own custom format, you can do the same thing with your own app without location permission.

## Don't Take It Too Far

If you take this approach, be careful that you don't go too far.  Don't just create a new beacon format and deploy many of thousands of devices that others will use for the same purpose in other apps.  If Google sees that a new beacon format is being widely used, they will just add it to their block list causing all that magic to stop.

And you absolutely should abide by Android's guidelines for declaring "neverForLocation" on your Bluetooth permission.  Google says you should be able to "strongly assert that your app never derives physical location from Bluetooth scan results".  If you are making this declaration with the intent of figuring out the user's latitude and longitude (instead of just *relative proximity* to a Bluetooth device without a known fixed location) then you are breaking the rules.  Please don't do that.

## Addendum: Sample Code

### iOS Details

You can see a simple reference app with the iOS code needed to do this here: https://github.com/davidgyoung/not-a-beacon-ios

To add this to your own app:

1. Add these entries to your Info.plist:

    ```
    <key>UIBackgroundModes</key>
    <array>
        <string>bluetooth-central</string>
    </array> 
    <key>NSBluetoothAlwaysUsageDescription</key>
    <string>This app uses Bluetooth to make magic happen when become near or far.</string>
    <key>NSBluetoothPeripheralUsageDescription</key>
    <string>This app uses Bluetooth to make magic happen when become near or far.</string>
    ```

2. Start scanning for a custom GATT Service UUID -- pick your own, but our example uses "4c052726-cd97-4dde-9356-212cc1327a84"

    ```
    func startScanning {
        // This line below will automatically pop a Bluetooth permissions request dialog using the above usage description
        centralManager = CBCentralManager(delegate: self, queue: nil, options: [CBCentralManagerRestoredStateScanOptionsKey: "MyAppToMakeMagic"])
        /// You should really wait until you get the Bluetooth state callback for central.state == .poweredOn before starting scanning
        centralManager?.scanForPeripherals(withServices: [CBUUID(string: "4c052726-cd97-4dde-9356-212cc1327a84")], options: [CBCentralManagerScanOptionAllowDuplicatesKey: true])
        }
    
    ```

3. In order to auto launch in the background, make you CBCentralManagerDelegate implement this method:

    ```
    func centralManager(_ central: CBCentralManager, willRestoreState dict: [String : Any]) {
        log.i("Central manager is restoring state after a background wakeup")
    }
    ```
4. And add code to your AppDelegate's didFinishLaunching method:

    ```
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]? = nil) -> Bool {
        ...
        // Execute the code the re-initializes your CBCentralManager and sets up its delegate before this method ends
        startScanning()
    }
    ```

## Android Details (using Android Beacon Library)

You can see a full reference version of the Android code needed to do this, including setting up a transmitter [here]( https://github.com/davidgyoung/android-beacon-library-reference-kotlin/tree/notabeacon).

To implement this in your own app:

1. Add this to your build.gradle

    ```
    dependencies {
        ...
        implementation 'org.altbeacon:android-beacon-library:2.21.0-beta4'
    }
    ```

2. Add this to your AndroidManifest.xml

    ```
    <uses-permission android:name="android.permission.BLUETOOTH_SCAN" android:usesPermissionFlags="neverForLocation" tools:node="replace"/>
    ```

3. Start scanning for a non-beacon based on a custom GATT Service UUID -- pick your own, but our example uses "4c052726-cd97-4dde-9356-212cc1327a84"

    ```
    val notaBeaconLayout = "s:0-15=4c052726-cd97-4dde-9356-212cc1327a84,m:16-16=00,i:17-18,i:19-20,i:21-22,p:-:-59"
    val notaBeaconBeaconRegion = BeaconRegion("notaBeaconBeaconRegion", BeaconParser("notabeacon").setBeaconLayout(notaBeaconLayout), null, null, null)
    val beaconManager = BeaconManager.getInstanceForApplication(this)
    beaconManager.startMonitoring(notaBeaconBeaconRegion)
    beaconManager.startRangingBeacons(notaBeaconBeaconRegion)
    ```

4. If you want to start a transmitter, use code like this:

    ```
    val transmitter = BeaconTransmitter(this,  BeaconParser("notabeacon").setBeaconLayout(notaBeaconLayout))
    val notabeacon = Beacon.Builder().setId1("1").setId2("2").setId3("3").build()
    transmitter.startAdvertising(notabeacon)
    ```

5. Also add code to [request Bluetooth permission](https://developer.android.com/training/permissions/requesting) from the user before starting scanning.  This can be a bit verbose, so we won't cover it here.
