
# The Secrets of iOS Bluetooth Advertising in the Background

Developers of Bluetooth apps on iOS have long struggled with the platform's limited functionality
in the background.  In particular, iOS apps can't freely emit Bluetooth advertisements when they are not
in the foreground.  On iOS, being in the foreground means that the app has to be visible on
the screen with the screen turned on.  This limits full functionality to when people are actually interacting
with the app.

When an iOS app is in the foreground, it can emit an iBeacon advertisement and it can emit a GATT service UUID.
In the background, it can do none of these things.  But backgrounded iOS apps are allowed to host Bluetooth
services.  And because hosting a Bluetooth service means that you have to "advertise" that service to others,
Apple must do *something* to allow this.  That somehow is called the "Overflow Area".  And understanding how
it works unlocks a world of possibilities for sending and receiving Bluetooth advertising data between iOS apps
in the background.

## Overflow Area 101

Typically, when a Bluetooth LE peripheral advertises itself to others, it transmits a distinct Service UUID to
let others know it is there.  A standard advertisement for a 128-bit service UUID consists of a packet type 0x07
followed by a 128-bits of the service.  A service with UUID 00000000-0000-0000-0000-000000000039 has an
advertising packet that looks like this:

```
07 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

Simple right?

This is a Bluetooth standard.  An advertisement like that is supported on all platforms including iOS and Android.

Apple lets a foregrounded app emit an advertisement like above, but move it to the background and this no longer
works.  Instead, the advertisement is moved to the "Overflow Area" so it looks like this:

```
ff 4c 00 01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80
```

The above advertisement is no longer type 0x07 -- it is now type 0xff, which is a manufacturer advertisement type.  The
`4c 00`  bytes correspond to Apple's assigned manufacturer code of 0x004C by Bluetooth SIG.    The next 17 bytes are
manufacturer data, and this type of packet can be used anyway the manufacturer wants.   There is no standard here
once you get past the  `4c 00`.  A manufacturer advertisement can be used for anything the transmitter and receiver want.

But what do the rest of the bytes mean?  How does this advertisement work?  Here's how Apple's documentation describes it:

> Any service UUIDs contained in the value of the CBAdvertisementDataServiceUUIDsKey key that don’t fit in the allotted space go to a special “overflow” area. These services are discoverable only by an iOS device explicitly scanning for them.
> While your app is in the background, the local name isn’t advertised and all service UUIDs are in the overflow area.

[Reference](https://developer.apple.com/documentation/corebluetooth/cbperipheralmanager/1393252-startadvertising)

That's super vague.  Typical Apple! Always hiding implementations in proprietary code.

 To find out how this works, I had to reverse engineer it.  I wrote an [iOS app](https://github.com/davidgyoung/BackgroundAdvertiser) that sequentially
advertises service UUIDs starting with 00000000-0000-0000-0000-000000000000, then
00000000-0000-0000-0000-000000000001, etc.  I then used an [Android app](https://github.com/davidgyoung/AdvetiserAnalyzer) to detect these advertisements and print out the
patterns in the 17 data bytes of the overflow area as binary.

From this, I learned that the overflow area works this way:

1. The first manufacturer data byte is always 0x01.  This lets you know the manufacturer data is an "overflow area" advertisement
2. The next 16 bytes are a 128 bit bitmask.  Each service UUID you advertise will cause exactly one of those 128 bits to be set to 1. In the example shown above, the very last bit in the bitmask is set to 1.
3. There is a one-to-one mapping between a service UUID and the bit position it sets in this bitmask.  This is consistent across iOS devices.  Converting a service UUID to a bit position in the bitmask is some proprietary Apple hashing algorithm.
4. Because there are a huge number of possible 128-bit UUIDs -- 2^128 (about 10^38) -- multiple service UUIDs share the same bit position.

## How iOS Uses the Overflow Area

When an iOS device is scanning for Bluetooth services, it can specify the service UUID it is looking for like this:

```
let serviceUuid = CBUUID(string: "00000000-0000-0000-0000-000000000039")
centralManager?.scanForPeripherals(withServices: [serviceUuid],
                                   options: [CBCentralManagerScanOptionAllowDuplicatesKey: true])
```

That code will give a callback when a standard type 0x07 service advertisement for that UUID is seen:

```
func centralManager(_ central: CBCentralManager, didDiscover peripheral: CBPeripheral,
                      advertisementData: [String : Any], rssi RSSI: NSNumber) {
}
```

The same code will also give a callback if it sees an overflow advertisement with the corresponding bit in its bitmask set for
that service UUID **but only if the screen is on.**

Yes, scanning the overflow area service advertisements only works if the screen is illuminated on the receiving device.  It doesn't
matter if the app is in the foreground (visible) or if it is in the background with another app or the springboard visible.  The
phone doesn't even need to be unlocked.  If the screen is on, locked or not, a callback for the overflow advertisement will get delivered to an app scanning for it
repeatedly each time it is detected.

The above screen-on restriction is why full background to background Bluetooth data exchange are often considered impossible on iOS.   While such an exchange is possible when both apps are in the background, it is only possible if the device receiving the advertisement has the screen on.  Fortunately there are tricks that can make the screen go on temporarily.*

> * A common trick to make the screen go on is to send a local notification.  So long as the user is not in do not disturb mode
on iOS, this will cause the screen to illuminate for 10 seconds, and overflow advertisements to get delivered during that time. By periodically
forcing the screen on with this technique, an iOS app can discover services advertised from nearby backgrounded iOS apps.

## How iOS Handles Collisions

Since many service UUIDs share each bit position in the overflow area bitmask.  What happens if an iOS app is scanning for a
service UUID encounters another backgrounded iOS app that is advertising a different service UUID that uses the same bit position?

The answer is that iOS will give a scanning callback for the colliding but different service UUID.  This won't happen often. 
But programmers should realize that they may scan for their service only to get a callback for detecting a backgrounded iOS
device advertising a completely different service that just happens to collide in the overflow area's bitmask.

## Hacking the Overflow Area

Now that we know how the overflow area works, we can use it for all kinds of other things, **including  data exchange between backgrounded iOS apps.**

### Using Overflow Area on Other Platforms

This is easy to do.  Using the info described so far you can make an Android device (or other non iOS device) detect a service UUID of interest on a backgrounded iOS device.  Just write code that looks for any overflow area advertisement (`ff 4c 00 01`), look for the bit position a known service uses and adjust your detection code to verify that bit is set in the bitmask.  The scanning device can connect to the advertising device knowing it likely hosts the service of interest.

### Detecting Any Backgrounded iOS Service Advertiser

iOS apps are not allowed to receive non-iBeacon advertisements in the background unless they specify the particular service UUIDs they are looking for.  But knowing that there are only 128 bits in the overflow area bitmask, you can write an app that scans for every possible bit in the bitmask.  This will give a callback for any iOS device advertising a service in the background regardless of service UUID.  

### Background iOS Data Exchange

The overflow area can be manipulated programmatically to put any pattern you want (except all zeroes) into the 16 byte bitmask area.  These data can then be transmitted from a backgrounded iOS app.
The data can be received by iOS, Android and other devices, even in the background.  On iOS, the backgrounded reception does require that the screen be on.  But again, this can be forced periodically to do a quick read by sending a local notification.
Another limitation is that the overflow area is shared between all apps on the phone.  Any app that advertises a Bluetooth service in the background will set (usually one) bit in the bitmask.  A second app on the same phone has no way of knowing this is happening.  So while an app can guarantee a pattern of 1s is set in the bitmask, it cannot guarantee any 0s.  In practice, it is rare for an iOS device to be running any backgrounded apps advertising Bluetooth.  So while you will usually get all 0s in the bitmask in positions you do not set, this is not guaranteed.
Setting up data exchange is a bit tricky.  The key is to generate a table of 128 different service UUIDs known to occupy a distinct position in the bitmask.  Fortunately, I have already done that for you.  See the code in my [OverflowAreaUtils class](https://github.com/davidgyoung/BackgroundAdvertiser/blob/master/BackgroundAdvertiser/OverflowAreaUtils.swift).

Using this utility, an iOS receiver can  scan for 128 different UUIDs, one for each position in the overflow area's bitmask.  

````
centralManager?.scanForPeripherals(withServices: OverflowAreaUtils.allOverflowServiceUuids(),
                                        options: [CBCentralManagerScanOptionAllowDuplicatesKey: true])
````

When it gets a callback, the callback will provide a list of all UUIDs it found.  You can convert this list to a 128 bit number:

```
    func centralManager(_ central: CBCentralManager, didDiscover peripheral: CBPeripheral, advertisementData: [String : Any], rssi RSSI: NSNumber) {
        if (advertisementData[CBAdvertisementDataOverflowServiceUUIDsKey] != nil) {
            if let overflowIds = advertisementData[CBAdvertisementDataOverflowServiceUUIDsKey] {
                if let overflowIds = overflowIds as? [CBUUID] {
                    NSLog("Overflow Area bitmask as binary string: \(OverflowAreaUtils.overflowServiceUuidsToBinaryString(overflowUuids: overflowIds))")
                }
            }
        }
    }
```

On the transmission side, an iOS advertiser can generate a 128 bit number and then convert any set bits to a corresponding service UUID to be advertised:

```
// Binary for the ASCII sequence: "OverflowAreaWoot"
let binaryString = "01001111011101100110010101110010011001100110110001101111011101110100000101110010011001010110000101010111011011110110111101110100"
let adData = [CBAdvertisementDataServiceUUIDsKey :
              OverflowAreaUtils.binaryStringToOverflowServiceUuids(binaryString: binaryString)]
peripheralManager?.startAdvertising(adData)
```

The code shown above will transmit the 128-bit ASCII string, "OverflowAreaWoot" from a backgrounded iOS app to another backgrounded iOS app with the screen on.

Using code like above, you can effectively exchange anything you can fit in 128 bits of data between backgrounded  iOS apps in a single packet.  And because overflow advertisements are sent out at 1Hz, you can send more data by altering the advertisement in time.  You just have to make sure the receiving iOS device has the screen turned on to receive it.  

Oh, and don't forget, you cannot send 128 bits of zeroes.  If you don't advertise at least one service (for one bit position set to 1) no advertisement will go out.

### The Tragedy of the Commons

As discussed above, the overflow area is a shared resource between all apps.  Any app can set one or more bits.  No app can know what bits other apps on the phone have set.  

While the typical iOS device will have no backgrounded apps setting bits in the overflow area, this is not guaranteed.  Apps that want to reliably exchange data using the techniques described above might consider  logic to account for bit collisions with other apps.  Given that most apps using the overflow area for its intended purpose will typically not set more than one bit, it is possible to periodically alter the overflow area data exchange to shift the transmission left or right in the bitmask to avoid  collisions with a bit or two that are stuck in the on position by other apps.

But an app that manipulates the overflow area by setting multiple bits is effectively polluting a common resource.  It is "overgrazing the commons" as British economist William Forster Lloyd described.

This is no big deal if only one app does it per phone.

However, if two apps on the same phone try to use the data exchange technique at the same time, both will fail.  If you plan to use this, realize it will only work until some other app on the phone tries to do the same thing.















