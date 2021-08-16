---
title: Is That a Scanner in Your Pocket?
---

# Bluetooth Scanning With Embdded Systems

Since the invention of Bluetooth LE, the most common use case has been to use a large device like a phone or laptop to scan for and connect to small peripheral devices like earpieces, health trackers and various remote sensors.  But some of the more innovative projects turn this idea upside down, building small devices that do Bluetooth LE scanning themselves, looking for phones, laptops, cars, and even other small peripherals that happen to be nearby.

During the pandemic, countries like Singapore used small Bluetooth scanners like this to bring automated contact tracing to elderly residents without smart phones.   All kinds of other use cases are possible including making hardware devices aware of their environment and even giving them social awareness.

It's common knowledge in the tech community that while Andorid and iOS devices are usually used in Bluetooth LE central mode (scanning for and connecting to peripherals), they can also work in peripheral mode (advertising themselves so others may scan for them.) What tech folks less often realize is that many embedded platforms allow the same kind of dual operation.   A plastic package might be scanning you right now.

## Choose Your Foundation Carefully

If you want to build a product that relies on distributed Bluetooth scanning, you've got to pick your hardware platform wisely as it will be the foundation of your system.  Special care should be taken to figure out your budget for per unit hardware, up-front hardware design and manufacturing, software development, and power.

Below is a comparison of some of the different scanning platforms available, showing the code needed to set up scanning on each.

### iOS and Android

These platforms excel in that they require no custom manufacturing, they are easy to program, and they have relatively large built in batteries.  Android models may be quite cheap as well.  It's really hard to complete with existing manufacturing economies of scale.  Take advantage of this, and don't build your own hardware if you don't have to.

Of the two platforms, iOS is much less flexible for scanning.  You cannot control the scan rate, and unless your code is running with a visible app on the screen, the scan rate will be low.  If the screen is on, your battery drain will be terrible.   If you can live with these serious limitations, you can start a scan like this (using Swift):

```
let centralManager = CBCentralManager(delegate: self, queue:DispatchQueue.global(qos: .default))
centralManager.scanForPeripherals(withServices: nil, [CBCentralManagerScanOptionAllowDuplicatesKey: true])

...

// When devices are detected, this method will be called
func centralManager(_ central: CBCentralManager, didDiscover peripheral: CBPeripheral, advertisementData: [String : Any], rssi RSSI: NSNumber) {
  if let serviceData = advertisementData["kCBAdvDataServiceData"] as? [NSObject:AnyObject] {

    // TODO: do something with the advertisementData here

  }
}
```

Android, by contrary, gives you much more flexibility. You can scan with the screen off at higher rates.  All kinds of form factors are available and you can buy very cheap reference designs.  Just be sure to test them for quality before you go too far.  With Android starting a scan is like this (using Java):

```
BluetoothManager bluetoothManager = (BluetoothManager) context.getApplicationContext().getSystemService(Context.BLUETOOTH_SERVICE);
BluetoothAdapter bluetoothAdapter = bluetoothManager.getAdapter();
BluetoothLEScanner scanner = bluetothAdapter.getBluetoothLeScanner();
scanner.startScan(scanCallback);

...

ScanCallback scanCallback = new ScanCallback() {

  // When devices are detected, this method will be called
  @Override
  public void onScanResult(int callbackType, ScanResult scanResult) {

    // TODO: do something with the scanResult here

  }
  ...
};
```


### Raspberry Pi

A number of Raspberry Pi form factors are available with Bluetooth LE built in.  The hardware is quite cheap and requires no custom manufacturing unless you want a snazzy case than you can buy off the shelf.  Programming is almost as easy as on iOS and Android.  While the BlueZ Bluetooth stack is fiddly and not well understood, experienced Linux C programmers are not difficult to find and can get the job done.

The real drawback is power.  These devices are designed for a wired USB power supply.  If you can plug them into a wall, great.  If not, expect to pair them with a large capacity USB battery.  Battery consumption is much worse than an Android or iOS device -- neither the hardware nor Linux are optimized to save power. 

Here's how you can start a scan with BlueZ (using C):

```
le_set_scan_parameters_cp scan_params_cp;
memset(&scan_params_cp, 0, sizeof(scan_params_cp));
scan_params_cp.type  = 0x00;
scan_params_cp.interval = htobs(0x0010);
scan_params_cp.window = htobs(0x0010);
scan_params_cp.own_bdaddr_type  = 0x00; 
scan_params_cp.filter = 0x00; 
struct hci_request scan_params_rq = ble_hci_request(OCF_LE_SET_SCAN_PARAMETERS, LE_SET_SCAN_PARAMETERS_CP_SIZE, &status, &scan_params_cp);
ret = hci_send_req(device, &scan_params_rq, 1000);

le_set_scan_enable_cp scan_cp;
memset(&scan_cp, 0, sizeof(scan_cp));
scan_cp.enable = 0x01; 
scan_cp.filter_dup = 0x00; 
struct hci_request enable_adv_rq = ble_hci_request(OCF_LE_SET_SCAN_ENABLE, LE_SET_SCAN_ENABLE_CP_SIZE, &status, &scan_cp);
ret = hci_send_req(device, &enable_adv_rq, 1000);

uint8_t buf[HCI_MAX_EVENT_SIZE];
evt_le_meta_event * meta_event;
le_advertising_info * adv_info;
int len;
while ( true ) {
  int len = read(device, buf, sizeof(buf));
  if ( len >= HCI_EVENT_HDR_SIZE ) {
    meta_event = (evt_le_meta_event*)(buf+HCI_EVENT_HDR_SIZE+1);
    // When devices are detected, this event will be triggered
    if ( meta_event->subevent == EVT_LE_ADVERTISING_REPORT ) {
      uint8_t reports_count = meta_event->data[0];
      void * offset = meta_event->data + 1;
      while ( reports_count-- ) {
        adv_info = (le_advertising_info *)offset;
        // TODO: do something with the adv_info here 
      }
    }
  }
}
```

### Nordic Semiconductor 

The Nordic chips in the nRF52x family are  the most widely used Bluetooth chipsets for Internet of Things projects.  Unlike the solutions above, this is a raw Bluetooth chip that  allows custom programming.  The have an embedded ARM processor that can be programmed in C using the Nordic SDK.   While it is not terribly hard to find a programmer to do this work, it is much harder than for the options above.  Engineers that write this kind of software rarely speak English, even for the those few where it is their native tongue.  Expect to deal with some serious neckbeard types here.

While you can buy battery-powered Nordic development kits as circuit boards the size of a deck of cards, these are not cheap and  almost never are used in production.  Nordic chips are almost always used on custom hardware, meaning you must design and build your own printed circuit board with all the other computer components you need added on.  You'll have to build your own power supply and provide your own battery, too.  This requires serious hardware engineering effort, a fairly large up-front R&D cost, and a contract manufacturer.   But once it is done, the per-unit costs can be quite low.

Programming with the Nordic SDK is notoriously difficult due to sparse documentation that consists of a few sample applications, terse API docs, and Q&A forums filled with posts from frustrated developers.  Most samples deal with using Nordic chips as a peripheral (meaning it advertises itself to be scanned by a phone or a laptop.). But it is also quote possible to program a Nordic chip to do BLE scanning.  Like this (using C):

```
NRF_BLE_SCAN_DEF(scan);
...

ble_gap_scan_params_t scan_param = {
    .active        = 0, 
    .interval      = 4000,
    .window        = 4000,
    .timeout       = BLE_GAP_SCAN_TIMEOUT_UNLIMITED, 
    .scan_phys     = BLE_GAP_PHY_1MBPS,  
    .filter_policy = BLE_GAP_SCAN_FP_ACCEPT_ALL,
    .extended      = 0, 
    .report_incomplete_evts = 0, 
    .channel_mask = {0x00, 0x00, 0x00, 0x00, 0x00} 
};

nrf_ble_scan_params_set(&scan, &scan_param);
ret_code_t err_code = nrf_ble_scan_start(&m_scan);

// When devices are detected, this function will be called:
static void ble_evt_handler(ble_evt_t const *p_ble_evt, void *p_context) {
  ble_gap_evt_t * p_gap_evt = &p_ble_evt->evt.gap_evt;
  switch (p_ble_evt->header.evt_id) {
      const ble_gap_evt_adv_report_t *p_adv_report = &p_gap_evt->params.adv_report;
      const ble_data_t *adv_data = &p_gap_evt->params.adv_report.data;

      // TODO: do something with the adv_data here

      break;
  }
}
```

### ESP32

The ESP32 is the new kid on the block and an attractive alternative to Nordic for an embedded system.  It is a family of chips designed by Espressif systems based on the Xtensa microprocessor, and the most popular variants come with both Bluetooth LE and WiFi on board.   While designing a custom PCB based on the ESP32 still requires hardware engineering and custom manufacturing, for projects requiring WiFi connectivity the onboard WiFi radio simplifies this process.  In addition, the development kids are relatively cheap with a small form factor suitable for production in some situations.  These development kits require USB power from a wired source or a battery.

Finding a programmer may not be easier than with Nordic, but the platform's popularity with hobbyists means you may be able to find somebody younger and cheaper to do the work  than the crusty neckbeards who dominate Nordic development.

Starting a bluetooth scan is relatively simple.  Like this (using C):

```
static esp_ble_scan_params_t ble_scan_params = {
    .scan_type              = BLE_SCAN_TYPE_ACTIVE,
    .own_addr_type          = BLE_ADDR_TYPE_PUBLIC,
    .scan_filter_policy     = BLE_SCAN_FILTER_ALLOW_ALL,
    .scan_interval          = 4000,
    .scan_window            = 4000,
    .scan_duplicate         = BLE_SCAN_DUPLICATE_DISABLE
};

esp_err_t scan_ret = esp_ble_gap_set_scan_params(&ble_scan_params);

// When devices are detected, this function will be called:
static void esp_gap_cb(esp_gap_ble_cb_event_t event, esp_ble_gap_cb_param_t *param) {
    switch (event) {
        case ESP_GAP_BLE_SCAN_RESULT_EVT: {
            esp_ble_gap_cb_param_t *scan_result = (esp_ble_gap_cb_param_t *)param;
            switch (scan_result->scan_rst.search_evt) {
                case ESP_GAP_SEARCH_INQ_RES_EVT:
                    uint8_t *adv_data = scan_result->scan_rst.ble_adv;
                    int adv_length = scan_result->scan_rst.adv_data_len;

                  // TODO: do something with the adv_data here

                break;
            }
            break;
        }
    }
}
```

### BlueNRG

Like Nordic and ESP32 this is a programmable SoC that requires you to build your own PCB with supporting hardware.  This chipset is provided by ST Microelectronics, and is much less common.  It's lack of popularity is likely driven by its limited flexibility -- it provides little control over scans.   This chip has all the disadvantages of the Nordic and ESP32 competitors, but is much less common.  You probably won't choose to use this chip -- but you may find yourself doing so because somebody else has made that choice for you.  You are unlikely to find a programmer who has used this before, so find somebody with embedded programming experience and  pay for him or her to learn how to use it.

The platform lets you start a scan like this (using C):

```
uint8_t ret = aci_gap_start_observation_proc(
    0x4000, /* Scan window */
    0x4000, /* Scan interval *.
    0, /* Do a passive scan */
    0, /* Use public address */
    1, /* Ignore duplciates */
    0 /* Don't filter scan results */
);

// When devices are detected, this function will be called:
void hci_le_advertising_report_event(
    uint8_t advertisement_count,
    Advertising_Report_t *advertisements)
{
  for (int i = 0; i < report_count; i++) {
    Advertising_Report_t  advertisement = advertisements[i];
    // TODO: Do something with the advertisement here
  }
}
```

## Got the Power?

When it comes to Bluetooth scanning, any of the above platforms can get the job done.

But that scanning comes with a cost -- the battery.   Scanning is typically 10-100 times more energy intensive than advertising, mostly because the radio needs to be turned on only for tiny intervals in order to advertise, but for long periods of time to scan.  Powering up the bluetooth radio and leaving it on to scan will noticeably drain the relatively large battery your phone.  For a small embedded device, scanning will suck the life out of a coin cell before a day is out.  If you want to scan, be prepared to provide some real power: a large battery, a wired supply, or regular recharging via a manual procedure or a small solar array.

There are many options to deal with this, even if you have to rely on batteries.   You can save power by limiting scanning to specific times that are important.  You can convince users of your system to put your device on the charger.  And you can squeeze as much life out of your device as possible by fitting it with as big of a battery as you possibly can.

If you can work with one of the platforms above, and can find a way to deliver the power, the possibilities will be as limitless as the creativity you bring to the table.














