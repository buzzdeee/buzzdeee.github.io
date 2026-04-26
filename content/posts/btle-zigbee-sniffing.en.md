---
title: "Sniffing Invisible Signals: BTLE and Zigbee on OpenBSD"
subtitle: "Professional-grade wireless analysis with nRF51822 and nRF52840"
description: "Explore BTLE and Zigbee traffic on OpenBSD using nRF52840 hardware, Kismet, and Wireshark for real-time wireless analysis."
date: 2026-04-24T16:24:29+02:00
lastmod: 2026-04-24T16:24:29+02:00
draft: false
tags: ["OpenBSD", "Bluetooth", "Zigbee", "Thread", "Security", "Kismet", "Wireshark"]
categories: ["Tutorials"]

images: [ "/images/sniffing_invisible_signals_btle_zigbee.png", "/images/nRF_devices.jpeg", "/images/Kismet_BTLE.png", "/images/Wireshark_BTLE_siffer_device_selection.png", "/images/Wireshark_BTLE_sniffing_in_action.png", "/images/Wireshark_802154_siffer_device_selection.png" ]

featuredImage: "/images/sniffing_invisible_signals_btle_zigbee.png"
featuredImagePreview: "/images/sniffing_invisible_signals_btle_zigbee.png"

toc:
  enable: true
seo:
  images: [ "/images/sniffing_invisible_signals_btle_zigbee.png", "/images/nRF_devices.jpeg", "/images/Kismet_BTLE.png", "/images/Wireshark_BTLE_siffer_device_selection.png", "/images/Wireshark_BTLE_sniffing_in_action.png", "/images/Wireshark_802154_siffer_device_selection.png" ]

---

In my previous posts, we've journeyed through the process of [Bridging the Gap: Bringing Modern Kismet to OpenBSD](/en/modern-kismet-on-openbsd/) and had some [Fun with RTL-SDR on OpenBSD](/en/fun-with-rtl-sdr/) tracking planes and smart meters. Today, I’m diving deeper into the invisible signals surrounding us by shifting our focus to **Bluetooth Low Energy (BTLE)** and **Zigbee**.

I've always been curious about what IoT devices around me are actually saying "behind my back" - fitness trackers, smart lightbulbs, and even toothbrushes are constantly "advertising" their presence. In this post, we will look at how to set up a professional-grade sniffing environment on **OpenBSD** using both the classic [Adafruit Bluefruit Sniffer (nRF51822)](https://www.adafruit.com/product/2269) and the modern [nRF52840 (specifically the nice!nano)](https://nicekeyboards.com/docs/nice-nano/).

---

## The Protocols: BTLE vs. Zigbee

Before we start, it is important to understand the landscape of the 2.4 GHz band. We'll be looking at two major players: **Bluetooth** and **Zigbee**.

### 1. Bluetooth: Classic vs. Low Energy (BTLE)

It is crucial to distinguish between "Normal" (Classic) Bluetooth and BTLE. They are actually two different protocols that share the same name and frequency band, but they are not cross-compatible.

* **Bluetooth Classic:** Designed for high-data-rate, continuous connections (like your wireless headphones). It’s power-hungry and stays "connected" to maintain throughput.
* **BTLE (Bluetooth Low Energy):** Designed for low power consumption and short bursts of data. It allows devices to run for years on a single coin-cell battery. It primarily uses a **Star Topology**, where a central device (like your phone) talks to several peripherals (like a heart rate monitor).

#### BTLE Versions
* **BTLE v4.x:** The foundation of modern IoT. It brought the "Low Energy" moniker to the mainstream. It is limited to 1 Mbps and has a shorter range.
* **BTLE v5.x:** The current standard. It introduced **2 Mbps** speeds for faster data transfer and **Coded PHY** for significantly longer range (up to 4x the distance).

### 2. Zigbee & Thread (IEEE 802.15.4)

Zigbee and Thread are the "industrial cousins" of BTLE. While Bluetooth is built for personal area networks, these protocols are designed for reliability and scale in home automation.

Their killer feature is the **Mesh Topology**, which allows every powered device (like a lightbulb) to act as a repeater, passing signals along to the next node. This creates a self-healing network that can cover an entire building without needing every device to be within range of the central hub.

* **Zigbee:** The established veteran. It is a full-stack protocol that defines everything from how the radio talks to how a "light bulb" object is structured. It is widely used in Philips Hue and IKEA TRÅDFRI systems.
* **Thread:** The modern newcomer. While it uses the same 802.15.4 radio layer as Zigbee, Thread is **IP-based** (IPv6). This means Thread devices can be addressed directly over the internet (via a Border Router) and forms the primary networking foundation for **Matter**.



* **Channels:** Unlike Bluetooth's frequency hopping, 802.15.4 protocols usually stay on a **fixed channel** (numbered 11-26).
* **Modulation:** Uses O-QPSK with DSSS (Direct Sequence Spread Spectrum), making it extremely resilient to background noise and interference from Wi-Fi.

---

### Comparison at a Glance

| Feature | BTLE | Zigbee | Thread |
| :--- | :--- | :--- | :--- |
| **Standard** | Bluetooth SIG | IEEE 802.15.4 | IEEE 802.15.4 (6LoWPAN) |
| **Topology** | Star | Mesh | Mesh |
| **Frequency** | 2.4 GHz (Hopping) | 2.4 GHz (Fixed) | 2.4 GHz (Fixed) |
| **IP Support** | No (Native) | No | **Yes (Native IPv6)** |
| **Max Speed** | 2 Mbps (v5) | 250 kbps | 250 kbps |
| **Philosophy** | "Connect to my phone" | "Build a reliable network" | "Every device is on the Internet" |


### Further Reading

**For Bluetooth:**
* [Bluetooth Core Specifications](https://www.bluetooth.com/specifications/specs/): The definitive (and massive) technical documents for every version.
* [Bluetooth LE Primer](https://www.bluetooth.com/bluetooth-le-primer/): A great high-level starting point for developers to understand the stack without reading 3,000 pages of specs.

**For Zigbee & 802.15.4:**
* [Connectivity Standards Alliance (Zigbee)](https://csa-iot.org/all-solutions/zigbee/): The official home of the Zigbee standard and the new Matter protocol.
* [Digi Knowledge Base: 802.15.4 vs Zigbee](https://www.digi.com/resources/standards-and-technologies/802-15-4-vs-zigbee): A fantastic technical breakdown of the layers between the raw radio standard and the Zigbee protocol.
* [Oscium: Zigbee and Wi-Fi Coexistence](https://www.oscium.com/training/zigbee-wifi-coexistence/): Essential reading to understand how to pick Zigbee channels that won't conflict with your home Wi-Fi.
* [Zigbee vs. Thread](https://www.smarthomeperfected.com/zigbee-vs-thread/): A comparison between the two home autmation protocols.
* [Using Wireshark to troubleshoot Thread networks](https://blog.golioth.io/using-wireshark-to-troubleshoot-thread-networks/): A practical guide to sniffing the emerging Matter/Thread ecosystem.

## The Hardware

{{< figure
    src="/images/nRF_devices.jpeg"
    alt="Adafruit Bluefruits and a few nice!nanos with soldered headers"
    caption="Adafruit Bluefruits and a few nice!nanos with soldered headers"
    class="ma0 w-75"
>}}

### 1. Adafruit Bluefruit Sniffer (nRF51822)

This device is based on the older Nordic **nRF51822** chipset. You can find these for about 30-40 Euro on **[eBay](https://www.ebay.de/sch/i.html?_nkw=Adafruit+Bluefruit+nRF51822&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF51822&toolid=10001&mkevt=1)**. It is a fantastic, compact entry-level tool. While it can still "see" most modern devices - since even Bluetooth 5 devices
usually broadcast their initial presence using legacy v4-compatible packets - it has a hard ceiling. It cannot follow connections that switch to the faster 2 Mbps mode, and it is blind to the modern "Extended Advertising" or "Long Range" packets introduced in v5.

**The "Friend" vs. "Sniffer" Identity Crisis:**
If you have both the Bluefruit LE Friend and the Bluefruit Sniffer, you'll quickly realize they are physically identical. Even `dmesg` won't help you te
ll them apart, as they both use the same USB-to-UART bridge:

{{< highlight text >}}
uslcom0 at uhub5 port 4 configuration 1 interface 0 "Silicon Labs CP2102N USB to UART Bridge Controller" rev 2.00/1.00 addr 3
ucom0 at uslcom0 portno 0: usb1.1.00004.0
{{< /highlight >}}

**The Blink Test:** When you plug them in, look at the LEDs. The **Sniffer** firmware blinks **Blue**, while the **Friend** firmware blinks **Red**.



### 2. nRF52840 (The Modern Powerhouse)
The **nRF52840** is the successor to the nRF51 series and supports the full **BTLE v5** suite as well as the **IEEE 802.15.4** standard. This makes it a "universal" IoT sniffer, capable of listening to everything from high-speed Bluetooth 5 to mesh networks like **Zigbee** and **Thread**. 

I am using the **nice!nano**, a ProMicro-compatible board often used in custom mechanical keyboards. It’s perfect for this task because it fits into a standard breadboard and features a built-in battery charger, should you want to take your sniffing rig mobile.

{{< highlight text >}}
umodem0 at uhub5 port 1 configuration 1 interface 0 "Adafruit Feather nRF52840 Express" rev 2.00/1.00 addr 3
umodem0: data interface 1, has no CM over data, has break
umodem0: status change notification available
ucom0 at umodem0: usb1.1.00001.1
{{< /highlight >}}

These nice!nano boards can also be found on [eBay](https://www.ebay.de/sch/i.html?_nkw=Adafruit+Bluefruit+nRF51822&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF51822&toolid=10001&mkevt=1) for just about 4 Euro. At this price point, it’s worth picking up a small handful of them. Having a few extra boards lets you dedicate one to each protocol—BTLE, or 802.15.4 saving you the hassle of constant re-flashing when you want to switch between different sniffing tasks.

You can find its documentation [here](https://docs.nordicsemi.com/bundle/ncs-latest/page/zephyr/boards/others/promicro_nrf52840/doc/index.html) and its [pinout/schematics here](https://nicekeyboards.com/docs/nice-nano/pinout-schematic).

#### Flashing the nice!nano
To turn this into a sniffer, we will use the firmware provided by the [Makerdiary nRF52840 ConnectKit](https://github.com/makerdiary/nrf52840-connectkit) repository.

1.  **Clone the Repository:**
    {{< highlight bash >}}
    git clone https://github.com/makerdiary/nrf52840-connectkit.git
    cd nrf52840-connectkit
    {{< /highlight >}}

2.  **Enter Bootloader Mode:** Connect the device via USB. Use tweezers to short the **RST** and **GND** pins twice quickly (or use a jumper if you've soldered headers).
3.  **Mount the Drive:**
    {{< highlight bash >}}
    doas mount /dev/sd1i /mnt  # Adjust device node as needed
    {{< /highlight >}}
4.  **Flash:** Choose your target protocol:
    * **For BTLE:** `doas cp ./firmware/ble_sniffer/nrf_sniffer_for_bluetooth_le_v4.1.1.uf2 /mnt/`

    {{< highlight bash >}}
    umodem0 at uhub5 port 3 configuration 1 interface 0 "ZEPHYR nRF Sniffer for Bluetooth LE" rev 2.00/2.04 addr 3
    umodem0: data interface 1, has CM over data, has no break
    umodem0: status change notification available
    ucom0 at umodem0: usb1.1.00003.1
    {{< /highlight >}}

    * **For Zigbee:** `doas cp ./firmware/nrf802154_sniffer/nrf802154_sniffer_v0.7.2.uf2 /mnt/`

    {{< highlight bash >}}
    umodem0 at uhub5 port 3 configuration 1 interface 0 "Nordic Semiconductor ASA nRF 802154 Sniffer" rev 2.00/3.06 addr 3
    umodem0: data interface 1, has CM over data, has no break
    umodem0: status change notification available
    ucom0 at umodem0: usb1.1.00003.1
    {{< /highlight >}}

5.  **Verify:** Check `dmesg`. It should either appear as "ZEPHYR nRF Sniffer for Bluetooth LE" or "Nordic Semiconductor ASA nRF 802154 Sniffer" depending on which sniffer firmware you flashed.

{{< admonition type="note" title="Restore Original Firmware" open=true >}}
When you need to restore the original firmware, download [s140_nrf52_6.1.1_softdevice.uf2](https://github.com/adafruit/Adafruit_nRF52_Bootloader/releases/tag/softdevice-uf2) from the official Adafruit repository.
{{< /admonition >}}

---

## Setting up the Software on OpenBSD

We will be using **Kismet** for real-time device discovery in our immediate environment, while leveraging **Wireshark** for more granular, detailed packet analysis.

{{< admonition type="info" title="OpenBSD Ports Available" open=true >}}
OpenBSD provides proper ports and packages for these tools, making the installation process straightforward and well-integrated with the base system.
{{< /admonition >}}

### 1. Installing Kismet
{{< highlight bash >}}
doas pkg_add kismet
doas usermod -G _kismet your_username
{{< /highlight >}}

### 2. Wireshark & Extcap Plugins
We need the Python dependencies and the plugins found in the Git repo we just cloned.

{{< highlight bash >}}
doas pkg_add wireshark py3-serial py3-psutil
doas usermod -G _wireshark your_username
mkdir -p ~/.local/lib/wireshark/extcap
{{< /highlight >}}

**Install the BTLE Plugin:**
{{< highlight bash >}}
cp -r tools/ble_sniffer/extcap/* ~/.local/lib/wireshark/extcap/
chmod +x ~/.local/lib/wireshark/extcap/nrf_sniffer_ble.*
{{< /highlight >}}

**Install the Zigbee (802.15.4) Plugin:**
{{< highlight bash >}}
cp tools/nrf802154_sniffer/nrf802154_sniffer/* ~/.local/lib/wireshark/extcap/nrf802154_sniffer.*
chmod +x ~/.local/lib/wireshark/extcap/nrf802154_sniffer.*
{{< /highlight >}}

{{< admonition type="warning" title="OpenBSD Serial Port Patch" open=true >}}
On OpenBSD, `py-serial` fails to detect the nRF sniffer automatically. To fix this, you must apply the patch below to `nrf802154_sniffer.py`. 

**Note:** This patch assumes your device is at `/dev/cuaU0`. If your sniffer attaches to a different port, you will need to adjust the script accordingly.
{{< /admonition >}}

{{< highlight bash >}}
diff --git a/tools/nrf802154_sniffer/nrf802154_sniffer/nrf802154_sniffer.py b/tools/nrf802154_sniffer/nrf802154_sniffer/nrf802154_sniffer.py
index afb7198..2e15ffe 100755
--- a/tools/nrf802154_sniffer/nrf802154_sniffer/nrf802154_sniffer.py
+++ b/tools/nrf802154_sniffer/nrf802154_sniffer/nrf802154_sniffer.py
@@ -78,7 +78,7 @@ class Nrf802154Sniffer(object):
     CTRL_ARG_LOGGER = 6
 
     # Pattern for packets being printed over serial.
-    RCV_REGEX = 'received:\s+([0-9a-fA-F]+)\s+power:\s+(-?\d+)\s+lqi:\s+(\d+)\s+time:\s+(-?\d+)'
+    RCV_REGEX = r'received:\s+([0-9a-fA-F]+)\s+power:\s+(-?\d+)\s+lqi:\s+(\d+)\s+time:\s+(-?\d+)'
 
     TIMER_MAX = 2**32
 
@@ -187,6 +187,7 @@ class Nrf802154Sniffer(object):
         """
         res = []
         res.append("extcap {version=0.7.2}{help=https://github.com/NordicSemiconductor/nRF-Sniffer-for-802.15.4}{display=nRF Sniffer for 802.15.4}")
+        res.append ("interface {value=/dev/cuaU0}{display=nRF Sniffer for 802.15.4}")
         for port in comports():
             if port.vid == Nrf802154Sniffer.NORDICSEMI_VID and port.pid == Nrf802154Sniffer.SNIFFER_802154_PID:
                 res.append ("interface {value=%s}{display=nRF Sniffer for 802.15.4}" % (port.device,) )
{{< /highlight >}}

### Device Permissions & Automation

While **Kismet** uses SUID root capture helpers and will "just work," **Wireshark** requires the `/dev/cuaU*` devices to belong to the `_wireshark` group to function without `doas`. 

On OpenBSD, you can automate this using **[hotplugd(8)](https://man.openbsd.org/hotplugd)**. This ensures that whenever you plug in your sniffer, the permissions are adjusted instantly.

#### 1. Identify your Sniffer
Run `usbdevs -v` to find the serial numbers of your nRF devices. You will need these for the configuration below.

#### 2. Create the Attach Script
Create `/etc/hotplugd/attach` (and `chmod +x` it). This script monitors for your specific hardware and maps the driver (e.g., `uslcom0`) to the corresponding device node.

{{< highlight bash >}}
#!/bin/sh

DEVCLASS=${1}
DEVNAME=${2}
# Configuration: Space-separated list of authorized serial numbers
SNIFFERS_SERIAL="SERIAL NUMBERS OF YOUR nRF DEVICES"

case ${DEVCLASS} in
0)
    # We only act on the hardware driver (e.g., uslcom0, uftdi0)
    # This prevents the 'ucom' race condition.
    sleep 1
    for serial in $SNIFFERS_SERIAL; do
        # Check if the driver currently attaching ($DEVNAME) is 
        # listed in the same block as our Sniffer serial.
        is_match=$(usbdevs -v | grep -B 5 "driver: $DEVNAME" | grep "$serial")

        if [ -n "$is_match" ]; then
            # Now we find the ucom child. On OpenBSD, ucom usually 
            # shares the same unit number as the parent hardware driver.
            # uslcom0 -> ucom0 -> /dev/cuaU0
            UNIT=$(echo "$DEVNAME" | tr -d -c '0-9')
            CUADEV="/dev/cuaU$UNIT"

            if [ -c "$CUADEV" ]; then
                GROUP=_wireshark
                chgrp "$GROUP" "$CUADEV"
                chmod 660 "$CUADEV"
                # Create a state file so we know this specific ucom was a sniffer
                touch "/var/run/hotplug_is_sniffer_$DEVNAME"
                logger -t hotplugd "SUCCESS: Hardware $DEVNAME ($serial) mapped to $CUADEV and changed group to $GROUP"
                break
            fi
        fi
    done
    ;;
esac
{{< /highlight >}}

#### 3. Create the Detach Script
Create `/etc/hotplugd/detach` (and `chmod +x` it) to revert the device nodes to their default state upon removal.

{{< highlight bash >}}
#!/bin/sh

DEVCLASS=${1}
DEVNAME=${2}
STATE_FILE="/var/run/hotplug_is_sniffer_${DEVNAME}"

case ${DEVCLASS} in
0)
    # Check if we previously marked this hardware as a sniffer
    if [ -f "$STATE_FILE" ]; then
        # Map driver name (uslcom0) to unit number (0)
        UNIT=$(echo "$DEVNAME" | tr -d -c '0-9')
        CUADEV="/dev/cuaU$UNIT"

        if [ -c "$CUADEV" ]; then
            # Revert to OpenBSD defaults
            chown root:dialer "$CUADEV"
            chmod 660 "$CUADEV"
            logger -t hotplugd "Reverted $CUADEV to root:dialer after detach."
        fi

        # Clean up the state file
        rm "$STATE_FILE"
    fi
    ;;
esac
{{< /highlight >}}

{{< admonition type="warning" title="Limitations of hotplugd" open=true >}}
`hotplugd` is not perfect. Even with the detach script, it may leave `/dev/cuaU*` devices assigned to the `_wireshark` group - for instance, if the host is shut down while the device is still plugged in.
{{< /admonition >}}

## Sniffing in Action

{{< admonition type="note" title="Hardware Limitations" open=true >}}
Please note that I currently do not have any active **Zigbee** or **Thread** devices in my local environment. Consequently, I am unable to provide live capture screenshots or show detected devices within Kismet and Wireshark for these specific protocols at this time.
{{< /admonition >}}

### Using Kismet
Plug in your device and launch Kismet pointing to the serial device (usually `/dev/cuaU0`).

**For BTLE:**
{{< highlight bash >}}
kismet -c nrf51822:type=nrf51822,device=/dev/cuaU0,name=btle_sniffer
{{< /highlight >}}

Access the Kismet web interfacce at `http://localhost:2501`.

{{< figure
    src="/images/Kismet_BTLE.png"
    alt="Kismet surveying BTLE devices"
    caption="Kismet surveying BTLE devices"
    class="ma0 w-75"
>}}

**For Zigbee (802.15.4):**
{{< highlight bash >}}
kismet -c nrf52840:type=nrf52840,device=/dev/cuaU0,name=zigbee_sniffer,channel_hop=false,channel=11
{{< /highlight >}}
*Note: Zigbee stays on one channel. 11, 15, 20, and 25 are common for smart home hubs.*


### Using Wireshark
If you've installed the plugins correctly, you will see "nRF Sniffer for Bluetooth LE" or "nRF Sniffer for 802.15.4" in the interface list.

#### BTLE sniffing

{{< figure
    src="/images/Wireshark_BTLE_siffer_device_selection.png"
    alt="select BTLE sniffer in Wireshark"
    caption="select BTLE sniffer in Wireshark"
    class="ma0 w-75"
>}}

{{< figure
    src="/images/Wireshark_BTLE_sniffing_in_action.png"
    alt="Wireskark BTLE sniffer in action"
    caption="Wireskark BTLE sniffer in action"
    class="ma0 w-75"
>}}

#### Zigbee sniffing

{{< figure
    src="/images/Wireshark_802154_siffer_device_selection.png"
    alt="select 802154 sniffer in Wireshark"
    caption="select 802154 sniffer in Wireshark"
    class="ma0 w-75"
>}}


**Pro Tip for Zigbee:** To see encrypted traffic, go to **Preferences > Protocols > Zigbee** and add the "Trust Center Link Key": `5A:69:67:42:65:65:41:6C:6C:69:61:6E:63:65:30:39`. For more regarding sniffing Zigbee see [Sniff Zigbee traffic](https://www.zigbee2mqtt.io/advanced/zigbee/04_sniff_zigbee_traffic.html)

---

## Summary

By combining the security-focused environment of **OpenBSD** with the versatility of the **[nRF52840](https://www.ebay.de/sch/i.html?_nkw=nRF52840+Nice!Nano&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF52840&toolid=10001&mkevt=1)** (and its predecessor, the **[nRF51822 Bluefruit Sniffer](https://www.ebay.de/sch/i.html?_nkw=Adafruit+Bluefruit+nRF51822&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF51822&toolid=10001&mkevt=1)**), you have a high-end protocol analyzer at your fingertips. From tracking Bluetooth advertisements to mapping out Zigbee mesh networks, the 2.4 GHz spectrum is no longer a black box.

While the [nRF51822](https://www.ebay.de/sch/i.html?_nkw=Adafruit+Bluefruit+nRF51822&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF51822&toolid=10001&mkevt=1) remains a classic for BLE-specific tasks, the [nRF52840](https://www.ebay.de/sch/i.html?_nkw=nRF52840+Nice!Nano&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF52840&toolid=10001&mkevt=1)'s multi-protocol support opens the door to modern IoT exploration that was previously difficult to access on a budget.

### Next Steps
The setup is ready, but the airwaves are quiet... for now. This sounds like a good excuse to dive into **Home Automation** soon, which will likely involve introducing several Zigbee and Thread-based devices to my lab. Once those are in place, I’ll follow up with a detailed article showing these devices being mapped and analyzed in real-time. Stay tuned!
