## The Mobile Location Permission Crackdown

Since beacon-enabled apps burst on the scene in 2013 both Android and iOS have been steadily cracking down on use of the technology in the background.
This has come in response to privacy concerns over apps that quietly use the technology to track users in the background.

This fall marks a big milestone in the way iOS and Android behave, with new restrictions showing up in the latest operating systems from
both platforms.  For iOS the changes are big and potentially alarming from the end-user perspective.



## iOS 13 Changes

Users who grant an app permission to always access location are periodically be presented with a warning dialog reminding them this is happening.  iOS 13 ow adds a map showing the specific locations
where the app read their location.

images/used-location-in-background.jpg
images/ios13-location-background-prompt.png

For many apps, the new dialog on the right is unnecessary alarming.  Think about an app that checks your location periodically to tell you where you last parked your car.  Even if the app doesn't share the location with anyone, it will need to
track wherever you drive just to know where you last were.  Such a dialog implies to the many users that this location information may be being transmitted off the phone for nefarious purposes, even if the app does none of these
questionable things.   The knee-jerk user response is to say, "why do they need to track me?" and then deny always permission.  They don't realize how the app works and why background location access is important.

In theory this is where the customizable text message comes in allowing an explanation of purpose.  But good luck writing such an explanation in one or two sentences that the user will understand.  The more likely response will
be to deny the permission, then complain that the app doesn't work when they later press the "find my car" button.  Here comes a one star review in the App Store!

Even if the user isn't suspicious about the app, seeing that dialog over and over gets annoying.  Unfortunately, the only way to shut it up permanently is by denying background location access by tapping "When in use".  Some folks will certainly pick that option for that reason.

Of course, some apps, do behave badly, which is precisely why Apple added this scary-looking dialog.  Honest app developers who need to track location in the background for legitimate purposes are collateral damage.

Unfortunately, this is not the only iOS 13 change.   When prompting the user for location permission for the app, iOS now offers a third option in addition to "Allow always" and "Allow when in use":  "Allow now".  The "allow now" option will give the app the ability to access your location on this app launch, but not the next one.  If the user selects "allow now", the next time the app is launched the user will be prompted again.

images/ios13-initial-location-prompt.png
images/ios10-location-promopt.png



But what about always access for getting location in the background?  Note that the new dialog above doesn't even give that option.  This is true even if the code that presented the dialog by specifically requesting always authorization with `locationManager.requestAlwaysAuthorization()`. The user has to grant one of these two options first, and once one is granted, the app may then request always authorization:

This two-step process is unfortunately cumbersome and will cause many users to be annoyed enough to deny always permission.


## Android 10 Changes

Changes in Android 10 are a bit of catch-up relative to iOS.  Android 10 now brings a new separate background location permission to the platform, and allowing the user to decide whether to grant "all the time" or "only while using the app" location permission.

| ![/images/android-9-location-prompt.png](/images/android-9-location-prompt.png) | ![/images/android-permission-dialog.png](/images/android-permission-dialog.png) |
|:--:|:--:|
| *Android 6-9* | *Android 10*|


Apple introduced the equivalent changes to iOS 8 back in 2014.  But for Android apps that were built to expect always having permission to track beacons in the background once the user gives initial consent, this is still a
big change.  Even if the app asks for background "all the time" permission, the user might only grant permission only when while using the app.   Apps must take care to expect this possibility and detect if the user has only granted "only while using the app" permission, ask the user to change the foreground-only location
permission.  The dialog below shows how that would look:


| ![/images/android-switch-to-always.png](/images/android-switch-to-always.png) |
|:--:|
| *Switching to All The Time Permission* |

<img src='/images/scanpermission.png' width="320px"/>

Note that unike iOS, the Android permission request dialogs don't show you a user-customizable justification section.  As a result, it is a good idea to send you own pop-up first, setting the expectations that you are about to ask for location permission, and explaining the proper reasoning.

## How We Got Here

For those curious about how we got to this point, here is a table that shows the evolution of location permission changes with operating system releases.


|Location Permission Restrictions|Android 4-5|Android 6-9|Android 10+|iOS 7|iOS 8|iOS 9-12|iOS 13+|
|                                | 2013      | 2015      | 2019      | 2013| 2014| 2016   | 2019  |
+--------------------------------+-----------+-----------+-----------+-----+-----+--------+-------+
|Dynamic Prompt Required?        | NO        | YES       | YES       | NO  | YES | YES    | YES   |
|Background/Foreground Separate? | NO        | NO        | YES       | NO  | YES | YES    | YES   |
|Background Use Warning Dialog   | NO        | NO        | NO        | NO  | NO  | YES    | YES   |
|Background Use Warning w/ Map   | NO        | NO        | NO        | NO  | NO  | NO     | YES   |
|Allow Once Offered              | NO        | NO        | NO        | NO  | NO  | NO     | YES   |
|Background Requires Second Step | NO        | NO        | NO        | NO  | NO  | NO     | YES   |

As you can see, Android 10 is currently where iOS 8 was in 2014.  Is that a bad thing?  Maybe -- if you think Apple's onerous new permissions process and frightening map dialog are a good thing.
For lemming-like users who tend to grant any permissions requested just to play a sketchy game app, Apple's approach might be a good thing.  But for thoughtful app developers trying to make apps that legitimately use location in the background, Apple's new restrictions are nothing less than a nightmare.
