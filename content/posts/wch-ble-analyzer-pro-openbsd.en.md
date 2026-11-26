---
license: "MIT"
title: "WCH BLE Analyzer Pro on OpenBSD: Three Channels in Parallel with Kismet and wch_capture"
subtitle: "Capture BTLE advertising on channels 37, 38, and 39 – using just a single USB port"
description: "Hands-on report on the WCH BLE Analyzer Pro on OpenBSD: BTLE advertising channels, Kismet data source, wch_capture, OpenBSD ports@ review, and PCAP/Wireshark analysis."
date: 2026-11-26T22:02:18+01:00
lastmod: 2026-11-26T22:02:18+01:00
draft: false
images:
  - "/images/WCH_BLE_OpenBSD.jpg"
  - "/images/wch-ble-analyzer-pro-front.jpg"
  - "/images/wch-ble-analyzer-pro-back.jpg"
  - "/images/Kismet_WCH_BLE_Data_Sources.png"
  - "/images/Kismet_WCH_BLE_in_Action.png"
  - "/images/WCH_BLE_Analyzer_Wireshark.png"

tags:
  - "OpenBSD"
  - "BTLE"
  - "BLE"
  - "Kismet"
  - "WCH"
  - "Sniffing"
  - "Ports"
categories:
  - "OpenBSD"
  - "Hardware"

featuredImage: "/images/WCH_BLE_OpenBSD.jpg"
featuredImagePreview: "/images/WCH_BLE_OpenBSD.jpg"

hiddenFromHomePage: false
hiddenFromSearch: false
twemoji: false
lightgallery: true
ruby: true
fraction: true
fontawesome: true
linkToMarkdown: true
rssFullText: false

toc:
  enable: true
  auto: true
  keepStatic: false
code:
  copy: true
  maxShownLines: 50
math:
  enable: false
  # ...
mapbox:
  # ...
share:
  enable: true
  # ...
comment:
  enable: true
  # ...
library:
  css:
    # someCSS = "some.css"
    # located in "assets/"
    # Or
    # someCSS = "https://cdn.example.com/some.css"
  js:
    # someJS = "some.js"
    # located in "assets/"
    # Or
    # someJS = "https://cdn.example.com/some.js"
seo:
  images:
    - "/images/WCH_BLE_OpenBSD.jpg"
    - "/images/wch-ble-analyzer-pro-front.jpg"
    - "/images/wch-ble-analyzer-pro-back.jpg"
    - "/images/Kismet_WCH_BLE_Data_Sources.png"
    - "/images/Kismet_WCH_BLE_in_Action.png"
    - "/images/WCH_BLE_Analyzer_Wireshark.png"
---

## So why bother? ;)

When I recently updated **[Kismet](https://kismetwireless.net/)** to the [latest release](/en/openbsd-ports-updating-spree-nov26/), I noticed the new data source [https://www.kismetwireless.net/docs/readme/datasources/bluetooth-wch-ble-analyzer-pro/](https://www.kismetwireless.net/docs/readme/datasources/bluetooth-wch-ble-analyzer-pro/). Since the data source built without any trouble, curiosity got the better of me and I had to order one – available, for example, from [https://www.olimex.com/Products/RISC-V/WCH/WCH-BLE-Analyzer-PRO/](https://www.olimex.com/Products/RISC-V/WCH/WCH-BLE-Analyzer-PRO/).

## Why three channels at once make all the difference

In Bluetooth Low Energy (BLE), there are 40 channels in the 2.4 GHz band (0 through 39), but for device discovery and connection setup the **three advertising channels** are crucial: **channel 37 (2402 MHz)**, **channel 38 (2426 MHz)**, and **channel 39 (2480 MHz)**. That is where devices broadcast their advertisements, respond to scan requests, or initiate connections.

A classic single-chip BLE sniffer has to hop back and forth between these channels. That is fine for a quick overview, but whenever packets are short-lived or connection timing gets interesting, you simply miss frames on whichever channel is not currently being monitored. If you want to keep all three advertising channels in view at the same time with conventional sniffers, you would need **three separate sniffers on three USB ports** – complete with cable clutter and scarce ports on your machine.

The **WCH BLE Analyzer Pro** solves this neatly: inside are **three CH582F RISC-V chips** paired with a **CH334 USB hub**. Each chip can handle a channel so all three advertising channels can be monitored in parallel – and it takes **only a single USB port** on the computer, with even enough room on the back of the hub to plug in another device.

{{< figure src="/images/wch-ble-analyzer-pro-front.jpg" alt="WCH BLE Analyzer Pro front panel with USB port" caption="WCH BLE Analyzer Pro front panel with USB port" class="ma0 w-75" >}}
{{< figure src="/images/wch-ble-analyzer-pro-back.jpg" alt="WCH BLE Analyzer Pro rear panel with antennas" caption="WCH BLE Analyzer Pro rear panel with antennas" class="ma0 w-75" >}}

{{< admonition type="info" title="Just one USB port on the host – plus a port on the back of the hub" open=true >}}
Instead of three separate sniffers on three USB ports, the WCH BLE Analyzer Pro uses only a single host port and still leaves room at the built-in USB hub on the back for another device. Especially on a laptop or compact test box, that keeps things pleasantly tidy.
{{< /admonition >}}

---

## Kismet, wch_capture, and OpenBSD

### Getting started right away with Kismet

Once the data source is built cleanly, you can quickly start a `pcapng` capture with Kismet. For initial experiments with debug output, a dedicated log directory comes in handy:

{{< highlight shell >}}
mkdir -p ble-logs && kismet -T pcapng -p ble-logs -t ble-test --debug
{{< /highlight >}}

This saves BLE packets directly in `pcapng` format via Kismet so you can inspect them further afterward.

Thanks to `libusb`, the data sources for all three channels are detected automatically. Just enable them in Kismet and off you go.

{{< figure src="/images/Kismet_WCH_BLE_Data_Sources.png" alt="Kismet wch-btle-mcu Data Sources" caption="Kismet wch-btle-mcu Data Sources" class="ma0 w-75" >}}
{{< figure src="/images/Kismet_WCH_BLE_in_Action.png" alt="WCH BLE Analyzer Pro BTLE monitoring in action" caption="WCH BLE Analyzer Pro BTLE monitoring in action" class="ma0 w-75" >}}

### `wch_capture` and clean `.pcap` files

Alongside Kismet, I came across [https://github.com/xecaz/BLE-Analyzer-pro-linux-capture](https://github.com/xecaz/BLE-Analyzer-pro-linux-capture). The tool provides a straightforward CLI to control PHY selection, filters, channel selection, custom 2.4G parameters, and output to `stdout` or a `.pcap` file:

{{< highlight text >}}
wch_capture                                                                                                                                                                          
Nothing to do – use -v and/or -w FILE.pcap
WCH BLE Analyzer PRO Linux Capture tool by Xecaz 2026!

Usage: wch_capture [OPTIONS]

Options:
  -v            Print packets to stdout
  -w FILE.pcap  Write PCAP (DLT 256, BLE LL + phdr)
  -p PHY        PHY: 1=1M (default), 2=2M, 3=CodedS8, 4=CodedS2
  -i ADDR       Initiator MAC filter  (AA:BB:CC:DD:EE:FF)
  -a ADDR       Advertiser MAC filter (AA:BB:CC:DD:EE:FF)
  -k KEY        LTK, 32 hex chars
  -K PASSKEY    BLE passkey (6-digit decimal)
  -2            Custom 2.4G mode (default: BLE monitor)
  -c CHAN       Channel 0-39: BLE adv 37/38/39 or 0=all (auto per MCU); 2.4G raw
  -A AADDR      2.4G access addr (hex, e.g. 8E89BED6)
  -C CRCINIT    2.4G CRC init (6 hex chars, e.g. 555555)
  -W WHITEN     2.4G whitening init (hex byte)
  -h            Show this help

Capture stops on SIGINT (Ctrl+C) or SIGTERM.
{{< /highlight >}}

On OpenBSD, `wch_capture` is easy to build. It only requires the `gmake` and `libusb-1.0` packages to be installed, and building is simply `gmake CFLAGS=-I/usr/local/include`.

From there you can run it right away, for example with live console output and writing to a `.pcap` file:

{{< highlight shell >}}
sudo /usr/local/bin/wch_capture -w output.pcap -v
{{< /highlight >}}

That produces nice `.pcap` files you can continue working with.

{{< admonition type="tip" title="Opening captures from Kismet and wch_capture in Wireshark" open=true >}}
Both captures from Kismet and `.pcap` files from `wch_capture` can be opened directly in Wireshark without needing extra plugins. That makes it easy to analyze raw captures from `wch_capture` in the familiar Wireshark UI.
{{< /admonition >}}

{{< figure src="/images/WCH_BLE_Analyzer_Wireshark.png" alt="BTLE traffic capture in Wireshark" caption="BTLE traffic capture in Wireshark" class="ma0 w-75" >}}

### Port review on the OpenBSD `ports@` mailing list

To keep `wch_capture` cleanly maintained and reproducibly buildable on OpenBSD, a port has been submitted for review to the OpenBSD `ports@` mailing list. Especially for specialized radio and capture tools, having a clean port on OpenBSD is worth its weight in gold.

---

## Conclusion

All things considered, it is a sturdy little piece of hardware, not overly expensive – and with Kismet and `wch_capture` almost a little too easy to use ;)

