
# Install and Run

## Case 1: I got a OpenWrt workspace

If you already have a working OpenWrt/Lede workspace, this is recommended.

### 1. Install the feeds.
Add the following line into "feeds.conf.default" (You will find it under the top dir of your workspace).

	src-git mtk https://github.com/Nossiac/mtk-openwrt-feeds;lede-17.01

then execute:

	scripts/feeds update -f mtk
	scripts/feeds install -a -p mtk

### 2. Select the packages you need.

After installing the feeds, try "**make menuconfig**":

	make menuconfig

You should be able to see all MTK WiFi drivers listed under "**MTK Properties -> Drivers**". Select the driver that fit your device.

**mtk-luci-plugin** is also recommended, it provides web UI that has built-in support for basic operations on MTK's wifi drivers. You need to install LuCi first.

### 3. Build and Install

:-)

This won't be an issue, since you are already a pro.

### 4. Bring it up!

Please not that: **The driver will not run automatically**.
This is for safety purpose. These modules are only tested with one or two actual products. It may not fit your device and possibly crash your system.

We take 7603 as an example, to bring it up manually:

	modprobe mt7603.ko
	ifconfig ra0 up

The wireless network is now working.

### Extra notes

1. Some drivers (like mt7612, mt7615) may have different interface names, like "rai0" or "wlan". You can find it out by `ifconfig -a` after `modprobe`.

2. If you have installed the "**mtk-luci-plugin**", you can bring up the wireless network with a single click on the "**Reload**" button.

3. **mtk-luci-plugin** also provides a command line tool `/sbin/mtkwifi`, which is quite handy to use: `mtkwifi [reload|start|stop]`

4. If you want the driver to run automatically, the easiest hack would be overwriting `/sbin/wifi` with `/sbin/mtkwifi` from **mtk-luci-plugin**. If you are a pro hacker, you can write your own rcS script.

## Case 2: I just want to try it.

### Download the prebuilt modules you need.

First of all, find out your hardware/software info and check if corresponding wifi module is available here: [http://nossiac.com/download/mtk-wifi-ko/](http://nossiac.com/download/mtk-wifi-ko/)

For example, if you are using a MT7621 router with mt7603 WiFi chip, and your kernel is linux-4.4.112, then you are expecting this:
[http://nossiac.com/download/mtk-wifi-ko/mt7603-for-mt7621-linux-4.4.112.ko](http://nossiac.com/download/mtk-wifi-ko/mt7603-for-mt7621-linux-4.4.112.ko)

If your device has access to the Internet, that would be quite simple:

	wget http://nossiac.com/download/mtk-wifi-ko/mt7603-for-mt7621-linux-4.4.112.ko -O /tmp/mt7603.ko

If you want to use `modprobe`, please remeber to put the driver under `/lib/modules/<your-linux-version>/`

### Bring it up!

Simply:

	modprobe mt7603
	ifconfig ra0 up

The wireless network is now working.

# Troubleshooting

### 1. the driver complains that some files were missing.

The driver usually loads 2 files during runtime.

* **mt76xx.dat** This is called "profile", which is actually the runtime configuration file for wifi driver. It has a list of "Key=Value" inside. If you build the module using **feeds**, the file will be ready, or you need to create one yourself.
* **e2p.bin / eeprom.bin**  This is a small binary file that contains a lot of HW parameters for the wifi chip. Usually the data is saved in a "flash" partition named "factory" on your device. It should be provided by the manufacture. By default OpenWrt will automatically detect the factory partition (if available), create the file(s) under /lib/firmware/ and move it in place:
```
root@OpenWrt:~# ls /lib/firmware/mt7662.bin -lah
-rwxr-xr-x    1 root     root       80.0K Jan 30 12:21 /lib/firmware/mt7662.bin
```
If the binary file is missing, you can try to extract it using `dd` and the following command: `dd if=/dev/$(grep -i '"factory"' /proc/mtd | cut -c 1-4) of=/lib/firmware/<FILENAME>.bin bs=1 count=512` (tested with Xiaomi Mi Router R3). If necceassry, use your stock firmware backup to recover the flash partition.

### 2. No wireless network can be found.
There are a lot of possibilities, you can dig some useful info following these steps:

* `lsmod | grep mt`, Check if driver loaded successfully.
* `ifconfig -a`, check if wireless interface created successfuly.
* `iwconfig` or `iw`, check if your SSID is there, otherwise the driver is not configured properly. Noe that `iwconfig` is a command from "**wireless-tools**" package, which is not selected by default.
* Check if you configured the SSID as "hidden".
* Check if the wifi is working on a channel that your device does not support.

### 3. My device cannot get an IP address.

OpenWrt's DHCP serves "br-lan“ only, make sure you added the wireless interface into ”br-lan".

	brctl addif br-lan ra0

This step is automatically done in **mtk-luci-plugin**.

# FAQ

### 1. Where can I get the driver source code?

I will not publish any driver source code here.

MTK's WiFi drivers do not have an open source licence. That is to say,
>The only legal way to get the source code is to sign the NDA and get the code from MediaTek.

However you can always find some code on the Internet, which is for sure leaked from MediaTek's vendors. It's your call on how to use it.

### 2. I need modules for kernel x.x.x

This is a reasonable request, I wish I can help if I can afford the time.

The original plan was to build the drivers with latest stable release of OpenWrt/Lede. As the project goes on, there will be more possibilities for various kernel support.

Please keep an eye on the project, that's an inspiration for the developers.

### 3. How can I configure the driver to ......

The **mtk-luci-plugin** already supports basic operations on MTK WiFi. It's a good example for developers to start their own project.

If you want more advanced features, you should either consult MediaTek, or pay some conpensation.

### 4. What's the relationship with the mt76 project?

In short, they are totally different implemantions.

The [mt76](https://github.com/openwrt/mt76) driver is GPL licensed. It was written from scratch by Felix@OpenWrt. It's much modernized. It follows the latest linux wireless architecture, and uses the latest kernel APIs.

The early stage of mt76's development once got supported by MTK, I guess that is the only connection between the two.

MTK's WiFi driver is non-open, aims for help MTK's clients and vendors make serious products. It runs not only under linux, but also under eCos and VxWorks (that's why its code looks weired). It's not friendly for beginners, as well as hackers.

### 5. Which one is better for me, MTK's driver or open source driver?

For end users and hackers, you should always try mt76 first. If you are not happy with the current quality, you can fall back on MTK's drivers.

If you are running a serious business, you should consider more, like stability, performance, advanced features, technical support, ...

Here's a short list you should take a look:

**Advantages of mtk drivers**

* Better performance. (if you configure it properly......)
* Full chips series coverage.
* More advanced features.
* Support eCos, VxWorks, ...
* Posible technical support from MediaTek.

**Disadvantages of mtk drivers**

* Does not support mac80211/cfg80211. Although cfg80211 is paritially supported in some drivers, but not quite ready for production.
* Does not support OpenWrt's uci configuration. That's why I wrote the "mtk-luci-plugin".
* Does not support the latest kernel. (depends on how this project goes)
* No GPL source code available.

### 6. I followed the steps from OpenWrt wiki or Google, but the driver does not work as expected.

Note that MTK's driver does not support **uci** configuration.You should either:

* Edit wifi profile directly. that how **mtk-luci-plugin** does it.

Or

* Use an adapter script or application that can translate **/etc/config/wireless** into MTK's wifi profile. Check [uci2dat](https://github.com/Nossiac/mtk-openwrt-feeds/tree/master/applications/uci2dat).

