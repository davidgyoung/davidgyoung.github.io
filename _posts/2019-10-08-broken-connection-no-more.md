## Hope for Android's Bluetooth LE Reliability Problem?

Android has long been known to have problems establishing and maintaining Bluetooth LE connections.  In a previous post,
I shared data showing that [Android has a stunning 20 percent failure rate](/2019/05/21/broken-connection) connecting to a GATT service under real-world conditions,
compared to just 3 percent on iOS.

It's not clear all the factors cause this reliability problem, but one known contributor is Android's use of a crazy setting for the Bluetooth link supervision timeout.  This
setting decides how long to wait after losing a contact with a BLE peripheral before Android gives up.  For Android 4.3-9.x, this was hard-coded to 20 seconds.
This is crazy long period compared with the iOS setting of 750 milliseconds.  Because it is hard-coded in Android, you cannot change this setting.  As a result, Android apps can't be notified of a broken BLE connection
until 20 seconds have passed.  What's worse, client code can't even try to reconnect until that 20 seconds is up.  For a user staring at a spinner on a screen,
20 seconds is an eternity.

The effect of this setting was noted by Andreas Schweizer in a [blog post](https://blog.classycode.com/a-short-story-about-android-ble-connection-timeouts-and-gatt-internal-errors-fa89e3f6a456) two years ago, and several folks have reported the problem to Google as a bug.

Not much changed until last month when Google released Android 10.  This is the first Android version to include a year-old [commit that reduces the supervision timeout from 20 seconds to 5 seconds.](https://android.googlesource.com/platform/packages/apps/Bluetooth/+/d32b2d46167122e876455ed70598b331fc692771%5E%21/#F0)
It's still hard-coded, and five seconds is still quite a long time compared with the 750ms on iOS, but it is at least an improvement that offers hope for more reliable Bluetooth LE connections on Android.

Using the same data set I used to calculated the 20 percent GATT connection failure rate on Android, very preliminary results with Android 10 devices offer hope for improvement.  The numbers so far are super small -- only 7 devices with Android 10 have
used the GATT service so far, but every one of those attempts was successful.  Cross your fingers that those very early numbers hold that trend.

Don't get to excited, though.  Even if Android 10 does offer significant reliability improvements, it will take years before that update gets to most users.  As we all know, Android has a huge problem with OEMs who rarely or never upgrade the operating system on their handsets.
If past experience is a guide to the future, it will be five years before 90 percent of Android devices have Android 10.  That's how long it took to reach the 90 percent distribution of Android 5.0 that exists today.

Five years is a long time to cross your fingers.  But with luck, by 2024, the worst of our Android Bluetooth connection reliability problems may be behind us.
