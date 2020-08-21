Using AE1000(or any Realtek 2870 based USB device) on old OS X (intel and powermac)
==============================================

To get a realtek based 2870 usb wifi adapter to work on a powermac (mine is an old g4 1.5 mac mini) or old intel mac, do the following:

- Found old version of RT2870 driver for OSX. Works on 10.3 - 10.6. Some of the ones available willl install but are only for intel macs.

    [Heres the driver for powerpc] (/files/RTUSB D2870-2.0.0.0 UI-2.0.0.0_2009_10_02.dmg)
    
    [Heres the driver for intel] (/files/MacDrivers4PandaUSBAdapterV1.7.dmg)

- Install the right driver from step 1.

- Once installed, plug in the ae1000 or whatever rt2870 device to usb, go to system profiler, explore the USB devices, and write down the product ID and vendor ID. These should be in hexidecimal.

- Convert those values to binary. This driver for whatever reason only works with binary format in it's plist file.

- Then change some permissions up:
    ```bash
    cd /System/Library/Extensions
    chmod -R 755 RT2870USBWirelessDriver.kext
    chown -R 0:0 RT2870USBWirelessDriver.kext
    ```

- Then edit plist file
    ```bash
    sudo vim /System/Library/Extensions/RT2870USBWirelessDriver.kext/contents/info.plist
    ```
    
    Look for the line(or you can copy a section):
    ```xml
    <key>Linksys - RT2870 - 2</key>
    ```
    
    You will need to modify the product and vendor id nodes to match the decimal equals from the 4th step.
    Save

- Touch extensions folder
    ```bash
    sudo touch /System/Library/Extensions
    ```

- Unplug usb device, restart, once booted up to aqua plug in the usb device