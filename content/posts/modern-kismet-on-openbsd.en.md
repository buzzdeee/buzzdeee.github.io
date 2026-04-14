---
title: "Bridging the Gap: Bringing Modern Kismet to OpenBSD"
subtitle: "Retiring the legacy ncurses port for a modern, multi-protocol RF sensor hub."
date: 2026-04-13T21:39:43+02:00
lastmod: 2026-04-13T21:39:43+02:00
draft: false
author: ""
authorLink: ""
description: "How a journey of code reduction, driver development, and a two-year wait finally brought modern Kismet to the OpenBSD ports tree."
license: "MIT"
images: [ "/images/modern-kismet.png", "/images/wifi-rf-equipment.jpeg" ]

tags: ["OpenBSD", "Kismet", "SDR", "Wireless", "C++", "Ports"]
categories: ["Tech", "Networking"] 

featuredImage: "/images/modern-kismet.png"
featuredImagePreview: "/images/modern-kismet.png"

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
  images: [ "/images/modern-kismet.png", "/images/wifi-rf-equipment.jpeg" ]
  # ...
---

For years, OpenBSD was stuck in a technical time-warp. While the rest of the world moved to Kismet "Newcore" -  with its powerful web-based UI, distributed capture, and multi-protocol support—the OpenBSD port remained anchored to the legacy ncurses version.

The core software, with some small patches, compiled fine, but there was a massive void: **no Wifi capture driver for OpenBSD.** To move forward and finally retire the ancient version, I had to build the engine: `capture_openbsd_wifi`.

## Navigating the `#ifdef` Labyrinth

When I started looking at the Linux capture code as a reference, I didn't just find complex logic—I found a pre-processor minefield. To support the "myriad ways" Linux manages network interfaces, the code is a dense jungle of:

* `#ifdef HAVE_LIBNM` (Network Manager)
* `#ifdef HAVE_LINUX_WIRELESS` (Legacy Wireless Tools)
* `#ifdef HAVE_LINUX_NETLINK` (Modern Netlink)
* `#ifdef HAVE_LINUX_IWFREQFLAG`

Maintaining that level of complexity is probably a full-time job. In stark contrast, OpenBSD’s "one way to do things" philosophy was a breath of fresh air. 

### The `cloc` Comparison: Linux vs. OpenBSD

The technical debt required for Linux versus the clean-slate implementation for OpenBSD is staggering. When I ran `cloc` (Count Lines of Code) on the drivers, the simplicity of the OpenBSD stack became undeniable:

| Driver Component | Files | Code Lines |
| :--- | :---: | :---: |
| **Linux Wifi Capture** | 10 | **5,479** |
| **OpenBSD Wifi Capture** | 1 | **1,075** |

By leveraging OpenBSD’s unified network stack, I achieved full parity with the legacy ncurses based version version of Kismet using **roughly 80% less code**. No `#ifdef` spaghetti required - just clean, functional code that talks directly to the kernel's wireless layer.

---

## The Hardware Rabbit Hole: BTLE and SDR

Once the Wi-Fi driver was stable, curiosity took over, and I began exploring other potential [datasources](https://www.kismetwireless.net/docs/readme/datasources/datasources/). I wanted to see how far I could push a "secure-by-default" OS into the world of RF monitoring, specifically looking at the **[RTL-SDR Blog V3](https://www.rtl-sdr.com/)** at that time, and the **[Adafruit Bluefruit LE Friend](https://www.adafruit.com/product/2267)** (running the Nordic Sniffer firmware). To get started with my experiments, I picked up both an **[RTL-SDR](https://www.ebay.de/sch/i.html?_nkw=RTL+SDR+BLOG&_sacat=0&_from=R40&_trksid=p2334524.m570.l1313&LH_TitleDesc=0&_odkw=RTL+SDR&_osacat=0&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=RTLSDR&toolid=10001&mkevt=1)** and an **[Adafruit Bluefruit](https://www.ebay.de/sch/i.html?_nkw=Adafruit+nrf51822+Bluefruit&_sacat=0&_from=R40&_trksid=m570.l1313&LH_TitleDesc=0&_odkw=Adafruit+BTLE+Bluefruit&_osacat=0&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Bluefruit&toolid=10001&mkevt=1)** on eBay.

This led to some satisfying technical victories:

* **BTLE without a BT Stack:** OpenBSD famously lacks a Bluetooth stack. However, since the `nrf51822` driver talks directly to the hardware via serial/USB, I was able to sniff BTLE traffic perfectly. I implemented the necessary patches to get the `nrf51` driver stable on OpenBSD, allowing for packet-level analysis on a platform that technically doesn't even "do" Bluetooth.
* **Fixing the Foundation:** My initial goal was to get **RTL-ADSB** working so I could track local aircraft. This required me to update the OpenBSD `comms/rtl-sdr` port from an aging release to **2.0.1**. Once that foundation was modernized, I was able to track planes in real-time, pulling ADS-B data directly into my dashboard.

## The Road to Upstream

Once the code was stable and the capture_openbsd_wifi driver was functional enough to replace the legacy version, I didn't want to just sit on a local branch. I submitted a few Merge Requests, and they were merged upstream. With the OpenBSD bits officially in the main repo, the foundation was there... and then I waited. ;)

## The Sprint to 2025

The wait for an official Kismet release took nearly two years. When the **September 2025** version finally dropped, I expected a simple port update. I was wrong. In those two years, Kismet's internal architecture had shifted enough to break the OpenBSD bits. 

After a few intense debugging sessions, I figured out what was needed to get the driver working again:

* **Adapting to Modern Kismet:** I had to refactor `capture_openbsd_wifi` to use the modern way to communicate with the Kismet server. 
* **Handling a "Hostile" OS:** During testing, I ran into a number of runtime issues and crashes. These only seemed to occur on OpenBSD - a testament to the "hostile" nature of the OS when it comes to memory safety and strict execution.
* **Curiosity and `rtl_433`:** In my initial attempt, I had completely ignored the `rtl_433` capture driver. But this time around, curiosity got the better of me. That curiosity pushed me to port `rtl_433` to OpenBSD and work out some necessary patches to get the datasource fully functional.

## Location Tracking

No wireless survey is complete without location data. I tested an old **Garmin GPSMap 60CSx** as a GPS device, that I used in the early 2000's in Geocaching adventures, in conjunction with Kismet. Despite the age of the hardware, it worked perfectly with the new setup, providing reliable coordinate data for the captured packets. Compared to the early 2000, those devices are meanwhile quite cheap on [eBay](https://www.ebay.de/sch/i.html?_nkw=Garmin+GPSMap+60CSx&_sacat=0&_from=R40&_trksid=m570.l1313&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Garmin60CSx&toolid=10001&mkevt=1). When you pick up one on eBay, ensure the special serial cable is included. They're quite neat for wardriving, since you can also attach an external antenna. 

{{< figure
    src="/images/wifi-rf-equipment.jpeg"
    alt="Wifi and RF equipment"
    caption="WiFi dongle, RTL-SDR v3, Adafruit Bluefruit nRF51822 Sniffer, Garmin GPSMap"
    class="ma0 w-75"
>}}


## Conclusion

The gap is finally closed. OpenBSD users can now ditch the ncurses past and embrace a world-class wireless IDS. There are still more capture sources to evaluate and look at, some of which may or may not require further fixes to play nicely with the BSD stack, but the foundation is solid.

For anyone interested in the details of the journey, the upstreamed patches are here: [github.com/kismetwireless/kismet (buzzdeee)](https://github.com/kismetwireless/kismet/commits?author=buzzdeee).

Keep an eye out for follow-up posts, where I'll be showing more details about sniffing different kinds of RF on OpenBSD.

**Happy sniffing. The ports are live.**

