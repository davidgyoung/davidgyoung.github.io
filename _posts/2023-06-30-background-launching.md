---
title: Background Launching 
---

The most common way to start a mobile app is to tap on its icon on your phone.  The app's user interface springs to life, and you can interact with it for awhile before moving on to other things.  Once the app is not visible, it is considered to be in the background.  On both iOS and Android, apps in the background may continue running (with some restrictions.)

But it is also possible to launch an app automatically, going directly to the background without the app ever appearing on the screen.  Understanding the details of how this works is important for developers building apps that need to run passively in the background.  This is especially true because both iOS and Android can kill your backgrounded app at any time if memory or battery are low.  Even if you don't care to launch an app automatically, you might wish to resume an app's background tasks automatically after the operating system temporarily killed it due to low resources.

## Alternate Ways to Launch

There are a number of ways to launch apps in the background.  Here are a few examples:

iOS:

* CoreLocation region events (enter/exit geofence regions, enter/exit Bluetooth beacon regions.)
* CoreBluetooth events (device detection, connect/disconnect and other events.)
* CallKit events (incoming Voice over IP calls)
* Push Notification events (incoming push notification directed to your app)
* Periodic Update Events (typically daily)

Android:

* Bluetooth events (device detection)
* Call events (incoming calls of various types)
* Phone Boot
* Power Connected
* Periodic Job Scheduler Events (every ~15 minutes max)

## iOS Launch Sequence

Most iOS apps implement a custom `AppDelegate` class which is the central point for receiving global events within the app.  Regardless of how an app is launched,  it always triggers AppDelegate's `didFinishLaunchingWithOptions` method:

```
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

        NSLog("App just launched!")
    }
}
```

After this method completes, additional callback methods may be fired, depending on what triggered the launch.  And if there is a root view (something most apps have), the app will attempt to start that up, too.  (Although if the app was launched in the background, it will not be actually shown on the screen.)

The details of creating any views on app launch depend on the configuration of your app in XCode (for views shown by configuration) or based on code you put in the `didFinishLaunchingWithOptions` method above (for views shown programatically).  It is important to understand that on iOS, this same process tries to create the same views regardless of whether your app is launched in the foreground or the background.


## Android Launch Sequence

On Android, there is no commonly used equivalent of the iOS `AppDelegate` to receive launch and other central events.  Android apps more typically have a a number of `Service` and `BroadcastReceiver` declarations declared in AndroidManfest.xml to handle background events.  These might look something like this:

```
<manifest package="com.example.myapp"
          xmlns:tools="http://schemas.android.com/tools"
          xmlns:android="http://schemas.android.com/apk/res/android">
    ...      
    <application android:icon="@mipmap/icon" android:label="@string/app_name" android:theme="@style/AppTheme">

        <service android:enabled="true" android:exported="false" android:name=".MyService"/>
        <receiver android:name=".MyBroadcastReceiver" android:exported="false"/>

        <activity android:name=".MainActivity" android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
```

However, Android also has what is known as an `Application` class that is used by all apps even if they do not customize it.  The `Application` class always gets a call to the  `onCreate` method as the very first callback to app code during the launch process, both for foregrounded and backgrounded apps.  While it is not as common for Android developers to make a custom version of this, you can do so by simply adding a `android:name=` element to the declaration to the `<application>` tag in AndroidManifest.xml, and provide a corresponding Kotlin or Java implementation for this class.  

Here's the manifest change:

```
<manifest package="com.example.myapp"
          xmlns:tools="http://schemas.android.com/tools"
          xmlns:android="http://schemas.android.com/apk/res/android">
    ...      
    <application android:name=".MyApplication" android:icon="@mipmap/icon" android:label="@string/app_name" android:theme="@style/AppTheme">
    ...

    </application>
</manifest>

```

And here is the class definition:

```
class MyApplication: Application() {
    ...
    override fun onCreate() {
        super.onCreate()
        Log.d(TAG, "App just launched!")
    }
}
```

The above `onCreate` method will get called whenever and however your app gets launched, even before any `Service`, `Activity`, or `BroadcastReceiver` gets created.  It is super useful for centralizing app logic in a similar way to iOS, especially from the background.

Unlike iOS, Android typically does not directly start views from the `Application` class.  This happens from `Activity` classes, which are created by the operating system *after* the `Application` class.  Whether or not an `Activity` gets created or not depends on how the app is launched.  If you launch by tapping on the app's' icon, Android creates an `Intent` that starts up the `Activity` with the `android.intent.action.MAIN` and `android.intent.category.LAUNCHER` declarations in your manifest.  If the app is not launched this way, then that `Activity` won't get started up at all.

Android always uses Intents to start up app components, and in addition to `Activity` instances, they can trigger `Service` and `BroadcastReceiver` instances.  But the `Application` instance is special -- there is one and only one of them, and it always gets created before anything else.  The very first time you start an `Activity`, `Service` or `BroadcastReceiver` when the app is not yet running, an `Application` is created, then the Intent target gets created (if it doesn't already exist).  The second time you do so, no new `Application` gets created.  In this way, the Android lifecycle is a bit more complex than iOS.

Expert's note:  'For the sake of completeness, it is worth mentioning that there are rare instances where two copies of a single Android app can be running simultaneously with two Application instances. This typical case is for apps that use special Service declarations in their manifest, indicating that the services must run in a separate process.  If you aren't doing that, it is safe to assume your app will always have exactly one Application instance.


## iOS and Android Launch Differently

Note their is a fundamental difference between iOS and Android background launches.  On iOS, the normal views (either `UIViewControlller` instances or `SwiftUI` instances) get created, and your logic inside those views executes just as if your app was launched by tapping on the icon.  The only difference is that the views exist only in the background, and are invisible to the user.

On Android background launches normally do not create views.  Any code that you want to execute based on a background launch should not be inside an Activity, but inside an `Application`, `Service` or `BroadcastReceiver`.


## App Suspension and Destruction

On both iOS and Android launched apps continue to exist in memory even they are no longer visible on the screen (or never have been).

On iOS, apps will typically be "suspended" by the operating system a few seconds after they go to the background, unless you make some declarations and do some tricks to get background running time.  But suspended doesn't mean the app is destroyed.  It will stick around in memory with the same `AppDelegate` instance in case the user starts interacting with it again or an event fires to give it more running time.  Only if memory gets low, the user kills the app, or the phone restarts is the app destroyed.  At that point, the `AppDelegate` instance is gone and the whole process has to start all over again.

On Android, apps are not quickly suspended in the background, although if the phone is motionless with the screen off, "Doze" mode will shut down the CPU and the app won't get any running time until the phone wakes up. Doze mode aside, code in `Activities` and `Services` can continue to run, although newer versions of Android will destroy them after about 10 minutes in the background (foreground services are a notable exception).  

Even if the Android operating system decides to kill app components after extended background time, the `Application` object is typically not destroyed along with `Activity`, `Service` and `BroadcastReceiver` instances, which and continues to exist.  The only time the `Application` object is destroyed is on full app destruction -- this happens in low memory or battery conditions.  This can also happen in other cases due to manufacturer-specific app killers in closed-source forked variants of Android -- something particularly problematic with Chinese OEMs like Huawei, RedMi and OnePlus.


## Background Launching Example - iOS Location Beacon

Starting up an app in the background on iOS can be tricky -- you may have to declare specific background modes in the Info.plist and obtain specific runtime permissions from the user first.  For Geofences and Bluetooth Beacons to wake an app, for example, you have to get "always" location permission from the user.  

If you have all of the above set up, you can then use the `CoreLocation` framework to monitor a beacon region:

```
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
            
        let locationManager = CLLocationManager()
        locationManager.delegate = self
        locationManager.startMonitoring(for: CLBeaconRegion(uuid: UUID(uuidString: "2F234454-CF6D-4A0F-ADF2-F4911BA9FFA6"), identifier: "my region"))
        ...
    }

    func locationManager(_ manager: CLLocationManager, didDetermineState state: CLRegionState, for region: CLRegion) {
        if (state == .inside) {
            NSLog("App is detecting Bluetooth Beacons matching the identifier pattern in \(region)")
        }
        else {
            NSLog("App is NOT detecting Bluetooth Beacons matching the identifier pattern in \(region)")
        }
    }

```

In the above code, we start "monitoring" beacon regions using iBeacon -- looking for when beacons become visible ("inside region") and invisible ("outside region").  The code simply logs each time an inside/outside transition happens.

But the magical thing about iBeacon on iOS is that the transition between inside to outside or outside to inside can launch an app into the background.  This is true even if the user killed the app manually, or has rebooted the phone since the last time the app was used.

Here's the sequence:

1. The iOS operating system's Bluetooth radio detects the previously registered beacon identifier pattern in an over the air advertisement.  
2. The IOS operating systems's CoreLocation framework launches the app.
3. The app starts up, calling the `didFinishLaunchingWithOptions` method of the AppDelegate, which itself sets up the CoreLocation delegate for any queued callbacks from the launch.
4. CoreLocation calls the `didDetermineState` delegate method to let the app know about the changes in beacon detections.


## Background Launching Example - Android Location Beacon

Background startup on Android can be similarly tricky, requiring you to configure the Manifest properly and obtain specific runtime permissions from the user first.  For Bluetooth Beacons to wake an app, for example, you have to get `BLUETOOTH_SCAN` and `BACKGROUND_LOCATION` permissions from the user.  Once this is set up, you can scan for Bluetooth beacons like this:


```
class MyApplication: Application() {
    ...
    override fun onCreate() {
        super.onCreate()
        Log.d(TAG, "App just launched!")
        
        val filters = ArrayList<ScanFilter>()
        // Scan for any Bluetooth device with a specific hardware address.  
        val filter = ScanFilter.Builder().setDeviceAddress("aa:bb:cc:dd:ee:ff").build()
        filters.add(filter)
        val settings = ScanSettings.Builder().setScanMode(ScanSettings.SCAN_MODE_LOW_LATENCY).setCallbackType(ScanSettings.CALLBACK_TYPE_ALL_MATCHES).build()
        val scanPendingIntent = PendingIntent.getBroadcast(context,0,Intent(this, MyBroadcastReceiver::class.java), PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_MUTABLE)
        val bluetoothManager = this.getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager
        bluetoothManager.bluetoothAdapter.bluetoothLeScanner.startScan(filters, settings, scanPendingIntent)

    }
}

class MyBroadcastReceiver: BroadcastReceiver {
    override fun onReceive(context: context, intent:Intent) {
        val bleCallbackType = intent?.getIntExtra(
            BluetoothLeScanner.EXTRA_CALLBACK_TYPE,
            -1
        ) 

        if (bleCallbackType != -1) {
            val scanResults = intent?.getParcelableArrayListExtra<ScanResult>(BluetoothLeScanner.EXTRA_LIST_SCAN_RESULT)
            for (scanResult in scanResults ?: ArrayList()) {
              Log.d(TAG, "The app just detected a Bluetooth beacon: ${scanResult.scanRecord}")
            }
        }

    }
}

```

In the above code, we start scanning for all Bluetooth devices matching a specific hardware MAC address and logs whenever it is detected.  Note that the code to set this up is lower level than that on iOS, because Android has no native support For Bluetooth Beacons.  For easier setup that makes things look a lot more like on iOS, you can try my [Android Beacon Library](https://altbeacon.github.io/android-beacon-library/).  But I want to show the nitty gritty details here so you see how background launching works.  Internally that library does the equivalent of the above.

Just like on iOS, the code above will auto launch the app in the background on a new Beacon detection.

Here's the sequence:

1. The Android operating system's Bluetooth radio sees an advertisement with a Bluetooth hardware identifier matching a `ScanFilter` set up in the above code.
2. The Android operating systems's Bluetooth component creates an `Intent` based on the `scanPendingIntent` in the above code.  This Intent matches `MyBroadcastReceiver`, so Android knows to launch that component.
3. The app starts up, calling the `onCreate` method of the `Application` class.
4. Android  constructs `MyBroadcastReceiver` and calls the `onReceive`  method to let the app know about the changes in beacon detections.

## Keeping the App Running in the Background

As discussed before, both iOS and Android have limits to how long apps can run in the background.   Even though your app is launched into the background, it may only get a few seconds of run time after the launch before it is suspended.  Extending this run time is possible.  Using advanced techniques, you can keep apps running in the background pretty much forever.  Those details are a topic for another post.

## What Launched My App?

If you have multiple events launching your app in the background things can get confusing.  During debugging you may wonder, "what launched my app?"   How you figure this out is quite different between iOS and Android.  Because it is much simpler on iOS, let's start there.

On iOS, you can simply check the options dictionary in `didFinishLaunchingWithOptions`:

```
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    if let launchOptionKey = launchOptions?.keys.first {            
    if launchOptionKey == .location {
            NSLog("This app was launched by CoreLocation.")
        }
        NSLog("Launch option info is: \(String(describing: launchOptions?[launchOptionKey]))")
    }
    else {
        NSLog("This app was launched manually by the user")
    }
    ...
```

You can see the full list of `UIApplication.LaunchOptionsKey` values [here](https://developer.apple.com/documentation/uikit/uiapplication/launchoptionskey).  This documents some of the several ways that iOS apps may be launched into the background.

Also note that the LaunchOptions dictionary also include both keys and values.  The values may give more specifics about why the launch type indicated by the key happened (e.g. a specific Geofence entry/exit for location, a specific URL being handled by your app, etc.)  Details of the information you get vary by launch type.

## Determining App Launch Reason on Android

On Android, figuring out why an app was launched is not so simple.  The `Application#onCreate` method has no equivalent to the iOS LaunchOptions mentioned above. In fact, there are no public APIs that you can use to see why the app was launched from inside this method.

This is true because of the differences in Android's architecture, which organizes apps via `Activity`, `Service`, and `BroadcastReceiver` components.  Any of these three component types can be started up via in Android `Intent`, and if the parent app is not already running the `Application` is created first and `Application#onCreate` is called.  Only after this method finishes executing does Android start up the actual component being launched.

If you have an app that creates these three types of components and want to know which started up the app in the `Application` class, you can add code to these components to **tell** the `Application` class how it was launched in a separate method that executes **after** `onCreate`.  The example below shows how this might be done.  Note that a new custom `onComponentStart` method in your Application class will log each time an app component starts up, and indicate whether this launched the entire app, or whether the app was already started when the component started.  Again, this new `onComponentStart` call will be *after* `onCreate`, because that is just how Android works.


```
// Modify your application class like this:
class MyApplication: Application() {
    var launchComponent: Any? = null
    fun onComponentStart(component: Any, intent: Intent?) {
        var componentStartType = "started after previous app launch"
        if (launchComponent == null) {
            componentStartType = "launched app"
            launchComponent = component
        }
        // Component launched app: MainActivity with intent Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] flg=0x10000000 cmp=org.altbeacon.beaconreference/.MainActivity }
        // Component started after previous app launch: MainActivity with intent Intent { flg=0x10000000 cmp=org.altbeacon.beaconreference/.MainActivity }
        Log.d(TAG, "Component $componentStartType: ${component.javaClass.simpleName} with intent: $intent ")
    }
    ...    
}

// add this to each Activity or Service
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        (this.application as MyApplication).onComponentStart(this, intent)
        ...
    }
    ...
}

// add this to each Broadcast Receiver
class MyBroadcastReceiver: BroadcastReceiver {
    override fun onReceive(context: context, intent:Intent) {
        (this.application as MyApplication).onComponentStart(this, intent)
        ...
    }
}
        
```

Take care that if you app uses third party libraries that expose other Activity`, `Service`, and `BroadcastReceiver` components, then the above changes won't be enough -- you'll also need to modify those third party components to do the same.
