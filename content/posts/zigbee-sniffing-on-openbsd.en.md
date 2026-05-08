---
title: "Zigbee Sniffing on OpenBSD: Diving Deeper with the TI CC2531"
subtitle: "A follow-up on Zigbee and BTLE sniffing, featuring the TI CC2531, upstream Kismet patches, and whsniff on OpenBSD."
date: 2026-05-08T21:30:33+02:00
lastmod: 2026-05-08T21:30:33+02:00
draft: false
author: ""
authorLink: ""
description: "Fixing Kismet mutex bugs, patching whsniff, and finally capturing 802.15.4 traffic on OpenBSD."
license: "MIT"
images: [ "/images/sniffing_802154_openbsd.png", "/images/ticc2531.jpg", "/images/kismet_zigbee.png", "/images/wireshark_802154.png" ]

tags: ["Zigbee", "Sniffing", "Kismet", "Wireshark", "CC2531", "802.15.4"]
categories: [ "Tech" ]

featuredImage: "/images/sniffing_802154_openbsd.png"
featuredImagePreview: "/images/sniffing_802154_openbsd.png"

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
  images: [ "/images/sniffing_802154_openbsd.png", "/images/ticc2531.jpg", "/images/kismet_zigbee.png", "/images/wireshark_802154.png" ]
  # ...
---

In my [previous post](https://buzzdeee.reitenba.ch/en/btle-zigbee-sniffing/), I explored using the nRF52840 nice!Nano for dual-duty BTLE and Zigbee sniffing. While the BTLE side worked like a charm, the Zigbee side remained stubbornly silent in Kismet and Wireshark.

To rule out firmware or hardware limitations, I decided to go with a classic: the **[Texas Instruments CC2531 USB Dongle](https://www.ebay.de/itm/404481824304?var=674447321503&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=TICC2531&toolid=10001&mkevt=1)**.

## The Hardware: TI CC2531

I picked up a [CC2531](https://www.ebay.de/itm/404481824304?var=674447321503&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=TICC2531&toolid=10001&mkevt=1) dongle on [eBay](https://www.ebay.de/itm/404481824304?var=674447321503&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=TICC2531&toolid=10001&mkevt=1), pre-flashed with the sniffer firmware and equipped with a "proper" external antenna. Unlike the [nice!Nano](https://www.ebay.de/itm/157602300188?amdata=enc%3AAQALAAAAoGfYFPkwiKCW4ZNSs2u11xC1d7FAa6flkF10AXmFbxjEfI9HUlmk3Monv69OxPg8gzKLTecE0Z2vlr4hjmU3PSz%2BOj0UTEToVcSBX7mUGUDjoywT1OguTvGKMjWykpZ6tqEnlcktfEpWSKxwXuXD1%2FTG%2BTU9F%2BrtgwVj7Ssv0ufYQ%2Br973tF29nUr5fy1WteZZFk%2FZrEkUWv6cM8oloBCF4%3D&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF52840&toolid=10001&mkevt=1), the CC2531 is a dedicated 802.15.4 radio, making it a more reliable baseline for debugging Zigbee traffic.

{{< figure
    src="/images/ticc2531.jpg"
    alt="Texas Instruments CC2531 with antenna"
    caption="Texas Instruments CC2531 with antenna"
    class="ma0 w-75"
>}}

## Fixing Kismet: Mutexes and Libusb

Kismet’s `ti_cc_2531` capture source needed some serious attention to run on OpenBSD. The OS was throwing `ABORT` signals due to improper mutex usage—specifically double-unlocks and attempts to unlock mutexes that were never locked. 

I also had to address a few `libusb` related issues surfacing on OpenBSD. For instance, `libusb_detach_kernel_driver()` isn't supported on OpenBSD, so the code needed to gracefully skip that step rather than failing. Additionally, I added `libusb_ref_device` calls to ensure the device reference count stays accurate while in use.

**The patches were merged upstream today!** For anyone interested into the details, here's the [PR](https://github.com/kismetwireless/kismet/pull/604)

With that, kismet finds the device, and can detect Zigbee devices.

{{< figure
    src="/images/kismet_zigbee.png"
    alt="Kismet detecting Zigbee devices on OpenBSD"
    caption="Kismet detecting Zigbee devices on OpenBSD"
    class="ma0 w-75"
>}}

## Bridging to Wireshark: whsniff

Kismet is great for discovery, but for deep packet inspection, Wireshark is the gold standard. To get the data from the CC2531 into Wireshark, I looked at [whsniff](https://github.com/homewsn/whsniff).

Initially, `whsniff` is geared towards Linux. However, with a tiny patch (see my [Pull Request #25](https://github.com/homewsn/whsniff/pull/25)), it now compiles and runs on OpenBSD. This allows you to pipe the output directly into Wireshark:

{{< highlight bash "linenos=table, hl_lines=1" >}}
# Example of piping whsniff into wireshark on OpenBSD
doas whsniff -c 11 | wireshark -k -i -
{{< /highlight >}}

This setup successfully allows me to analyze 802.15.4 network traffic in real-time.

{{< figure
    src="/images/wireshark_802154.png"
    alt="Wireshark sniffing IEEE 802.15.4"
    caption="Wireshark sniffing IEEE 802.15.4"
    class="ma0 w-75"
>}}

{{< admonition tip "Wireshark Zigbee Hints" >}}
Getting the traffic into Wireshark is only half the battle. Here are two things that will save you a lot of headache:

1. **Decryption**: Zigbee traffic is usually encrypted. You can add the default **ZigBeeAlliance09** trust center link key in hex format to **Edit -> Preferences -> Protocols -> ZigBee -> Pre-configured Keys**.
   * **Key (Hex):** `5A:69:67:42:65:65:41:6C:6C:69:61:6E:63:65:30:39`
   * If that doesn't work, you'll need to grab the specific network key from your home automation controller (like Zigbee2MQTT).
2. **Handling Noise**: The 2.4GHz band is noisy. You will likely see many packets with a malformed FCS (Frame Check Sequence). To force Wireshark to decode these anyway, uncheck: **Edit -> Preferences -> Protocols -> IEEE 802.15.4 -> "Dissect only good FCS"**.
{{< /admonition >}}

## The nice!Nano Mystery

As for the [nice!Nano/nRF52840](https://www.ebay.de/itm/157602300188?amdata=enc%3AAQALAAAAoGfYFPkwiKCW4ZNSs2u11xC1d7FAa6flkF10AXmFbxjEfI9HUlmk3Monv69OxPg8gzKLTecE0Z2vlr4hjmU3PSz%2BOj0UTEToVcSBX7mUGUDjoywT1OguTvGKMjWykpZ6tqEnlcktfEpWSKxwXuXD1%2FTG%2BTU9F%2BrtgwVj7Ssv0ufYQ%2Br973tF29nUr5fy1WteZZFk%2FZrEkUWv6cM8oloBCF4%3D&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF52840&toolid=10001&mkevt=1) I still haven't had success getting it to sniff 802.15.4 frames reliably on OpenBSD. It's possible the signal strength is an issue or the firmware isn't playing nice with my specific environment.

To investigate further, I’ve purchased a **[Sonoff Dongle M](https://www.ebay.de/itm/388971238285?amdata=enc%3AAQALAAAAoGfYFPkwiKCW4ZNSs2u11xBWFdbfFYp4er2kjyno9qmO593oK669JSSjjTgWvi6tTbapsCl%2FVp2mI58X9g82SgP81da1K3tWO4DD1pBdPF3sW8qHl6SAyOQrvVfMDGTMK23eQrjJoSroW1eTkSWk%2B2gMNy9psH35o8kjxRGrjbpxdfL5BH8uSE4Y44WKK5Swaf31ULdlZn1RqlWAhOr4IgM%3D&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Sonoff&toolid=10001&mkevt=1)**. My plan is to start my own home automation lab, which will allow me to place a Zigbee device in extremely close proximity to the nice!Nano to see if it's a sensitivity issue.

## Conclusion

The CC2531 remains a solid, inexpensive tool for the OpenBSD toolbox, especially now that the Kismet capture source is stabilized. While it's an older chip, the external antenna and dedicated firmware make it a great "known good" device to have on your desk.

I also sent a Kismet port update, as well as a new port for whsniff, to ports@ today for review, and I hope they can be integrated into the ports tree soon.

Stay tuned for the next update once the Sonoff hardware arrives!
