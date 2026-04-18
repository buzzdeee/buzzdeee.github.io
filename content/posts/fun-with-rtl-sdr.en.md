---
title: "Fun with RTL-SDR on OpenBSD: Planes, Meters, and Radio"
subtitle: "Sniffing packets from the airwaves on the most secure OS."
date: 2026-04-17T21:36:52+02:00
lastmod: 2026-04-17T21:36:52+02:00
draft: false
author: ""
authorLink: ""
description: "From tracking planes to reading smart meters, explore the versatile world of SDR on OpenBSD. No blobs, no fuss, just raw signal processing."
license: "MIT"
images: [ "/images/fun-with-rtl-sdr.png", "/images/RTL-SDR.jpeg", "/images/dump1090.png", "/images/kismet_rtl433.png", "/images/kismet_rtladsb.png", "/images/sdrpp.png" ]

tags: ["OpenBSD", "SDR", "RTL-SDR", "Radio", "Kismet"]
categories: ["Tech", "Networking"]

featuredImage: "/images/fun-with-rtl-sdr.png"
featuredImagePreview: "/images/fun-with-rtl-sdr.png"

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
  images: [ "/images/fun-with-rtl-sdr.png", "/images/RTL-SDR.jpeg", "/images/dump1090.png", "/images/kismet_rtl433.png", "/images/kismet_rtladsb.png", "/images/sdrpp.png" ]
  # ...
---

Following up on my previous post about [Bridging the Gap: Bringing Modern Kismet to OpenBSD](/en/modern-kismet-on-openbsd/), it is time to explore the hardware that makes much of this wireless exploration possible. If you have a spare USB port and a €50 RTL-SDR dongle, you can turn your OpenBSD box into a powerful radio scanner.

## What is SDR?

**Software Defined Radio (SDR)** shifts the heavy lifting of radio signal processing from dedicated hardware circuits to software. Traditionally, if you wanted to listen to a new frequency or decode a new protocol, you needed a specific physical radio. With SDR, the hardware is simply a "dumb" front-end that digitizes a chunk of the RF spectrum; your CPU handles the demodulation and decoding.

The **[RTL-SDR](https://www.rtl-sdr.com/)** is the gateway drug into this world. Originally designed as cheap DVB-T (digital TV) USB dongles, hackers discovered that the Realtek RTL2832U chip could be pushed into a "raw data" mode. This allows it to receive a wide range of frequencies - typically from **500 kHz to 1.7 GHz**.

The RTL-SDR Blog V3 and the most recent V4, as well as antennas and cables can be easily found on [eBay](https://www.ebay.de/sch/i.html?_nkw=RTL+SDR+BLOG&_sacat=0&_from=R40&_trksid=p2334524.m570.l1313&LH_TitleDesc=0&_odkw=RTL+SDR&_osacat=0&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=RTLSDR&toolid=10001&mkevt=1)

In the article I use a RTL-SDR Blog V3.

{{< figure
    src="/images/RTL-SDR.jpeg"
    alt="RTL-SDR Blog V3"
    caption="RTL-SDR Blog V3"
    class="ma0 w-75"
>}}

## The OpenBSD Setup

On OpenBSD, we start with `dmesg` to ensure the device is recognized as a generic USB device:

{{< highlight shell "linenos=table" >}}
ugen2 at uhub5 port 4 "Realtek RTL2838UHIDIR" rev 2.00/1.00 addr 4
{{< /highlight >}}

Once you have the `comms/rtl-sdr` package installed, your first step is running `rtl_test` to verify connectivity:

{{< highlight shell "linenos=table" >}}
sudo rtl_test
Found 1 device(s):
  0:  Realtek, RTL2838UHIDIR, SN: 00000001

Using device 0: Generic RTL2832U OEM
Found Rafael Micro R820T tuner
Supported gain values (29): 0.0 0.9 1.4 2.7 3.7 7.7 8.7 12.5 14.4 15.7 16.6 19.7 20.7 22.9 25.
4 28.0 29.7 32.8 33.8 36.4 37.2 38.6 40.2 42.1 43.4 43.9 44.5 48.0 49.6
[R82XX] PLL not locked!
Sampling at 2048000 S/s.

Info: This tool will continuously read from the device, and report if
samples get lost. If you observe no further output, everything is fine.

Reading samples in async mode...
lost at least 76 bytes
...
{{< /highlight >}}

### A Note on Async USB

When running `rtl_test`, you will likely notice these messages:
`lost at least 76 bytes`, `lost at least 68 bytes`...

This occurs because OpenBSD doesn't natively support the asynchronous USB protocol used by these drivers on other OSs. Instead, "Async" is wrapped around the synchronous USB protocol. While this causes minor sample loss at high sample rates, it is generally negligible for monitoring narrow-band signals or digital packets.

## CLI Fun: Monitoring the Neighborhood

### 1. Listening to FM Radio

Want to hear the local radio station? You can pipe the raw frequency data from rtl_fm directly into ffplay:


{{< highlight shell "linenos=table" >}}
rtl_fm -f 91.9M -M wfm -s 170k -r 44.1k | ffplay -nodisp -f s16le -ar 44.1k -
{{< /highlight >}}

### 2. Reading Weather Stations with  rtl_433 

The  rtl_433  utility is a Swiss Army knife for the 433.92 MHz ISM band. It decodes signals from weather stations, tire pressure sensors, and power meters.


{{< highlight shell "linenos=table" >}}
sudo rtl_433 -s 1024k
{{< /highlight >}}

{{< admonition type="warning" title="Notice on Sample Rates" >}}
While the RTL-SDR technically supports higher bandwidth, I found that using a sample rate higher than **1024k** led to instabilities and crashes on my **RTL-SDR Blog V3** under OpenBSD. If you are using the newer **V4** hardware, it may or may not be more stable, but starting at 1024k is a safe bet for reliability.
{{< /admonition >}}

{{< highlight shell "linenos=table" >}}
sudo rtl_433 -s 1024k
rtl_433 version 25.12 (2025-12-12) inputs file rtl_tcp RTL-SDR with TLS
Reading conf from "/etc/rtl_433/rtl_433.conf".
Found Rafael Micro R820T tuner
[SDR] Using device 0: Realtek, RTL2838UHIDIR, SN: 00000001, "Generic RTL2832U OEM"
[R82XX] PLL not locked!
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _
time      : 2026-04-14 20:59:35
model     : Nexus-TH     House Code: 196
Channel   : 2            Battery   : 1             Temperature: 11.60 C      Humidity  : 62 %
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _
time      : 2026-04-14 20:59:39
model     : Oregon-THGR810                         House Code: 203
Channel   : 0            Battery   : 0             Celsius   : 8.70 C        Humidity  : 72 %
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _
{{< /highlight >}}

In my testing, I immediately picked up a neighbor's sensors:
* **Nexus-TH:** Temperature: 11.60 C, Humidity: 62 %
* **Oregon-THGR810:** Celsius: 8.70 C, Humidity: 72 %

Whether you're living in a digital desert or a high-tech hub depends on just how many 'smart' upgrades you and the folks next door have embraced.

### 3. Tracking Planes with `dump1090`
Aircraft broadcast their position and altitude via **ADS-B** on 1090 MHz. Running `dump1090` turns your terminal into a radar screen:


{{< highlight shell "linenos=table" >}}
sudo dump1090
Found 1 device(s):
0: Realtek, RTL2838UHIDIR, SN: 00000001 (currently selected)
Found Rafael Micro R820T tuner
Max available gain is: 49.60
Setting gain to: 49.60
Exact sample rate is: 2000000.052982 Hz
Gain reported by device: 49.60
*5d4cadbbcf5037;
CRC: cf5037 (ok)
DF 11: All Call Reply.
  Capability  : Level 2+3+4 (DF0,4,5,11,20,21,24,code7 - is on airborne)
  ICAO Address: 4cadbb

*8d4cadbb586f963f3468ea17a936;
CRC: 17a936 (ok)
DF 17: ADS-B message.
  Capability     : 5 (Level 2+3+4 (DF0,4,5,11,20,21,24,code7 - is on airborne))
  ICAO Address   : 4cadbb
  Extended Squitter  Type: 11
  Extended Squitter  Sub : 0
  Extended Squitter  Name: Airborne Position (Baro Altitude)
    F flag   : odd
    T flag   : non-UTC
    Altitude : 21225 feet
    Latitude : 73626 (not decoded)
    Longitude: 26858 (not decoded)

*20000db9733fa9;
CRC: 733fa9 (ok)
DF 4: Surveillance, Altitude Reply.
  Flight Status  : Normal, Airborne
  DR             : 0
  UM             : 0
  Altitude       : 21225 feet
  ICAO Address   : 4cadbb
...
{{< /highlight >}}

### Deep Dive: Understanding the "DF" Codes
If you are looking at the `dump1090` output and wondering what **DF 17** or **ICAO Address** actually means, you are scratching the surface of Mode S transponder theory. 

The **Downlink Format (DF)** is the first 5 bits of a message that tells the receiver how to decode the rest. For example:
* **DF 11**: The "All Call" reply where a plane announces its unique 24-bit ICAO address.
* **DF 17**: The "Extended Squitter"—this is the core of ADS-B, containing GPS position, altitude, and velocity.

If you want to learn how to decode these bits by hand or understand the math behind the **Compact Position Reporting (CPR)** (which is why you often see "Latitude not decoded" until two consecutive packets are received), I highly recommend these resources:

* **[The 1090 Megahertz Riddle](https://mode-s.org/1090mhz/)**: An incredible open-access book by Junzi Sun that explains Mode S and ADS-B decoding in a very readable way.
* **[The Radar Tutorial](https://www.radartutorial.eu/13.ssr/sr24.en.html)**: Christian Wolff's definitive guide to radar technology and SSR/Mode S message formats.
* **[RTL-SDR.com](https://www.rtl-sdr.com/)**: The definitive blog for news on new hardware (like the V4 dongle) and community projects.

The output gives you real-time data on flight paths, altitudes, and ICAO addresses. You can even open `http://localhost:8080/` in your browser to see a web-based map of the planes currently in your airspace when using the `--net` parameter.
{{< highlight bash >}}
dump1090 --interactive --net
{{< /highlight >}}

{{< figure
    src="/images/dump1090.png"
    alt="dump1090 web interface"
    caption="dump1090 web interface tracking planes"
    class="ma0 w-75"
>}}


## GUI and Advanced Monitoring

### Kismet Integration
As mentioned in my last post, Kismet can use the RTL-SDR as a source for non-WiFi signals. 

* **To capture IoT/Meters:** `kismet -c source=rtl433-0:type=rtl433`

{{< figure
    src="/images/kismet_rtl433.png"
    alt="Kismet 433 meters"
    caption="Temperature meters detected with Kismet rtl433 datasource"
    class="ma0 w-75"
>}}

* **To capture Planes:** `kismet -c source=rtladsb-0:type=rtladsb`

{{< figure
    src="/images/kismet_rtladsb.png"
    alt="Kismet ADSB"
    caption="Planes detected with Kismet ADSB datasource"
    class="ma0 w-75"
>}}

### Coming Soon: SDRpp and Gqrx
While the command line is great, sometimes you want a visual "waterfall" to see where the signals are. 

I've recently taken a stab at getting **SDRpp** (a modern, fast SDR software) working on OpenBSD. It's a work in progress, but it promises a much more fluid experience than some of the older tools. Similarly, I'm looking into the state of the **Gqrx** port to provide a more traditional desktop radio interface.
Those and updates of other SDR related ports, you can find in my [mystuff](https://github.com/buzzdeee/mystuff/tree/main/comms) GitHub repo.

### Summary & State of the Ports

What is the current state of SDR on OpenBSD?

* **Available Now:** Excellent CLI support via `rtl-sdr`, `rtl_433`, and `dump1090`. 
* **Kismet:** Fully functional for ADS-B and ISM band monitoring.
* **The "GNURadio" Elephant:** While `gnuradio` is in the ports tree, it is a massive dependency chain and can be hit-or-miss regarding performance and stability on OpenBSD due to complex threading requirements.
* **The Future:** Keep an eye out for **[SDRpp](https://www.sdrpp.org/)**. I am working on making this a viable option for OpenBSD users who want a modern, high-performance GUI.

{{< figure
    src="/images/sdrpp.png"
    alt="SDR++ on OpenBSD"
    caption="SDR++ on OpenBSD"
    class="ma0 w-75"
>}}

OpenBSD proves once again to be a great platform for hardware hacking, provided you’re willing to dive deep into the inner workings of the system and don’t mind overlooking the occasional lost data point!
