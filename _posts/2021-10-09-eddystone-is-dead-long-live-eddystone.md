---
title: Eddystone is Dead, Long Live Eddystone!
---


If you've used Google products for more a few years, you probably know to approach them with caution.  A Google product can work great and have a loyal following, but that doesn't mean the company won't [axe it with short notice like hundreds of other products](https://killedbygoogle.com), leaving users howling with frustration. 

For users of Google's Bluetooth beacon platform, 2021 was the year to howl.  In April, the company shut down their cloud-based system for managing your beacons and Google Play Services delivering associated content to third party Android and iOS apps.  The system was never hugely popular, but folks who built their apps and products on top of these services were thrown under the bus by the silicon valley giant.

## Eddystone is Not Deprecated

This abandoned system is often confused with Google's Eddystone Bluetooth beacon standard.  Many people mistakenly think the Eddystone format itself was deprecated by Google.  Not so.  This open beacon standard was released in 2015 to compete with Apples' proprietary iBeacon standard (long ago [reverse engineered and published](http://www.davidgyoungtech.com/2013/10/01/reverse-engineering-the-ibeacon-profile)) and the open source [AltBeacon standard.](https://github.com/AltBeacon/spec)  

Beacon formats are called "standards" because they are just simple blueprints that anybody can implement.  Think of it like a Phillips screwdriver.  While any one company can stop making screws or tools that meet the Phillips screwdriver standard, everybody else can just keep making and using them.  So it is with Eddystone and its Eddystone-UID, Eddystone-TLM, Eddystone-URL variants.  

Eddystone and AltBeacon beacons formats are in the public domain (they have open source license from Google and Radius Networks, respectively) and nobody can stop folks from using them.   And to be clear, Google never even suggested people should stop using them.  To this day, Google continues to [publish the open Eddystone standards on their Github account](https://github.com/google/eddystone).  The only thing they have deprecated are their beacon cloud platform that worked with not just Eddystone but also iBeacon and AltBeacon.  

## Eddystone Without Google Services

Bluetooth beacons, Eddystone or otherwise, continue to work fine on Android phones.  Eddystone beacon formats not only work fine without Google's services, they are even more valuable, more stable, and more secure.    That's because you don't have to worry about Google sharing your beacon data or throwing your company under the bus when they shut off their service.  Any time you built a system that relies on third party services like the Google Beacon Platform, you are exposing yourself to a huge risk.  If that company abandons the service or changes the terms in some unacceptable way, you have to rebuild from scratch.

Back when Google was first developing Eddystone, I was working from Radius Networks (which had developed the open source AltBeacon standard) and we had several collaborative meetings with Google's Eddystone team.  I quickly realized that Google was using the beacons as a vehicle to push people into using their cloud services, probably so that they could use and monetize third party beacon network themselves.  To me, this seemed like a terrible idea.

At the time, my boss told me that this was the end of the [Android Beacon Library](https://altbeacon.github.io/android-beacon-library/), my open-source SDK that had become the de-facto tool on Android -- Google, he said, was taking over.  I smiled silently thinking of Google's short attention span.  Six years later, the Google behemoth has thrown in the towel, but the Android Beacon Library (developed largely by one guy) is still going strong.  Oh, and did I mention my Android Beacon Library still supports Eddystone, too?

## The Death of Eddystone-EID

There is one Google beacon format that is dead for most people -- Eddystone EID.  This is special format that uses a rotating crypto hash of the beacon identifier to make it harder for other people to spoof a beacon or freeload by using its transmission without permission.  It relies on a piece of server software called a "trusted resolver" to convert the scrambled identifier transmitted over the air to an identifier that is consistent and usable.   

While Eddystone-EID is an open standard, it is pretty much worthless without a "trusted resolver".  Because Google's deprecated cloud platform was the only publicly available "trusted resolver" for Eddystone-EID, if you want to use this format today, you have to build your own.  Building a trusted resolver is a non-trivial exercise -- I know because I had to build my own to test the Eddystone-EID beacons I was making before Google opened theirs up to the public.  I don't recommend building one yourself.

## Why Google Cloud Services Abandoned Eddystone

While Google engineers had some good ideas with Eddystone, some of the company's ideas were terrible.  The original idea of the Eddystone-URL was called the "physical web" meaning that you would connect the World Wide Web with the physical world by having beacon transmitters send out a URL to bring up on your phone.  A good example of where this might be useful is at a historical signpost.  Instead of a $2,000 bronze plaque with 200 words, you could just put up a $20 beacon and have it send a link to your phone, allowing you to bring up the Wikipedia page with as much info as you'd ever want to know.

But then Google's ad team got their hands on this idea.  They pushed a product called Google Nearby that would show beacon-based notifications directly on your Android phone, and bundled this with Google Play Services, which is installed on most  Android phones outside of China.  You can guess what happened next -- users were bombarded with spammy notifications about  shoe sales.   In the days where notification fatigue was still setting in, Google was on the cutting edge of annoying us.  Somebody in the company with half a brain finally shut down this service in 2018.  But come on, who couldn't see this coming?

As for the rest of Google's beacon cloud platform, nobody can say for sure why it finally got the axe.  Why does Google kill any of the hundreds of products it abandons?  There are certainly equipment and labor costs to keeping beacon services going -- operations needs to keep the servers up and support needs to answer developer questions.   If Google Nearby had not been a big disaster and if the cloud platform had otherwise been more popular it might have had a chance.  But like at any big company, the final decision usually comes down to the lack of an executive willing to defend a product.  If nobody inside Google was willing to stand up for it, why should the rest of us care?

## Should You Still Use Bluetooth Beacons and Eddystone?

Yes!  Google's short attention span shouldn't influence the rest of us.  There are all new and exciting use cases for bluetooth beacons that haven't even been considered yet.   Not only are iBeacon and AltBeacon formats great to use in the future, most Eddystone formats are too.  

Eddystone-UID, in fact, has some niche advantages over both AltBeacon and iBeacon.  When making iOS apps, the iBeacon format is almost always the best choice as it is detected very quickly.  But iBeacon has the disadvantage of only having four usable bytes (the major and minor) for data transfer on iOS.   The AltBeacon format gives you much more space for data transfer, but it cannot be detected by a backgrounded app.   Eddystone-UID, however, both larger space for data transfer, and it is able to be detected in the background on iOS.  Because of these advantage, I regularly use Eddystone-UID on new projects where data transfer in the background on iOS is important.

Just because Google can't find a use for Eddystone doesn't mean that you can't.  The Phillips screw company has long since moved on from their namesake screw patent from 75 years ago.  But their screw standard remains the most popular in the world today.  Likewise, the Eddystone standard will remain in use outside Google for many decades top come.












