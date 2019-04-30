When Google released the first Android phone back in 2010, one of the primary selling points to the geekiest among us was the openness of the platform.  Unlike Apple's closed-source walled garden known as iOS, Android was open source free for anyone to review and modify, and it was such more flexible in the way it could be used.  Far from open, iOS was by contrast riddled with maddening rules about what apps were and were not allowed to do.

Over the years, this openness was abused by many app developers.  Unlike iOS, Android allowed apps to spawn constantly running background services, burning the CPU, powering up the GPS, and transmitting loads of data over the cell connection.  The end result was much shorter battery life on Android devices.

Google began to get serious about combating these abuses starting with Android 6.  New features like Doze and App Standby helped tame battery usage, and in Android 8, Google finally outlawed the long-running background service.

For several phone manufacturers, especially in China, these changes were too late.   An especially large crop of abusive battery-draining apps hit the Chinese market well before Google's first battery saving changes in Android 6.  

Chinese manufacturers, many building low-end devices most susceptible to battery drain, took matters into their own hands.  They deeply forked their  Android implementations to go beyond a simple UI skin modifications and add in power management features that brute force killed apps running in the background for more than a few minutes, blocking them from starting again without a manual launch.

Just like Apple's iOS, these forked operating system changes are closed source.  But worse than iOS, the rules are either very poorly documented or not documented at all.  That makes building apps that work within the varying rules extremely difficult, essentially requiring reverse engineering each forked OS version.  And worse of all, the specific rules are different for each manufacturer.

Back before the OnePlus One burst on the scene, this was a problem only in the Chinese market.  But since then, Oppo the maker of the OnePlus line, as well as Huawei and Xiaomi have expanded sales to global markets. Finnish upstart HMD Global, building off of cheap Chinese reference designs and base firmware, has recently joined this party, selling phones under the Nokia brand, but with similar Chinese power saver badness built-in thanks to firmware supplied by contract manufacturers based in China.  Today, forked Android power savers are a global problem.

The folks at dontkillmyapp have done a great job of documenting restrictions as they become known.  You can check out their work here:  [https://dontkillmyapp.com](https://dontkillmyapp.com)

So why doesn’t this keep your email from synchronizing?  The three big Chinese manufacturers all work with app whitelists.  Apps that are on the whitelist (think Gmail, Spotify, Twitter, etc.) are exempt from restrictions that keep them from running in the background.  This is why you are still able to get email notifications on a Huawei phone.  But any app not lucky enough to be on the whitelist gets killed quickly after leaving the foreground.  It doesn't matter if you follow all of Google's open source Android rules and best practices for battery-friendly background activity: using Foreground Services and the Job Scheduler.  On these custom Android forks, your app will be killed anyway.

What can you do about it?  Nothing, short of warning your app users.  

You can detect if your app is running on a phone made by Oppo, Huawei or ZTE and tell users that certain background features provided by your app won't work.  You might tell the user that the features can be enabled by whitelisting the app.  You can even give instructions telling users exactly how to do this.

But *most* users will never do this.  They will  ignore warnings and instructions until the app fails to deliver promised functionality.  Then they will leave you a one star review in the Play Store with a terse line saying the app doesn't work.  

The brute force option is to block your app from being downloaded from the Google Play Store into devices made by these manufacturers.  A slightly less draconian alternative is to design a second "foreground only" experience for  the app that gets triggered when it is run on such Chinese devices.   But this approach runs the risk of  getting bad reviews if the Google Play listing makes promises for mainstream phones that can’t be delivered on Chinese phones.  This latter problem argues for a completely different app.

What you should **not** do is let your customers and your bosses think that your app can work the same way on these devices.   What you should also not do is bang your head against the desk trying to force your app to work on these devices when the manufacturer is secretly working against you.  Either approach is doomed to frustration and failure.


