---
title: "Two Weeks of OpenBSD Ports maintenance: LLVM 22 Fallout, SDR Stack Expansion, and Ruby Housekeeping"
subtitle: ""
date: 2026-06-04T22:32:06+02:00
lastmod: 2026-06-04T22:32:06+02:00
draft: true
description: "A retrospective on recent OpenBSD ports maintenance: resolving LLVM 22 fallout across GNUstep, expanding Software Defined Radio (SDR) tools, introducing GNUstep App Wrappers, and streamlining Ruby web scanners."
license: "MIT"
images: [ "/images/openbsd-ports-updating-spree-june4.png"]

tags: ["OpenBSD", "Ports", "GNUstep", "LLVM", "SDR", "Ruby", "Security"]
categories: ["Tech", "Porting"]

featuredImage: "/images/openbsd-ports-updating-spree-june4.png"
featuredImagePreview: "/images/openbsd-ports-updating-spree-june4.png"

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
  images: [ "/images/openbsd-ports-updating-spree-june4.png" ]
  # ...
---

It has been a busy two weeks in the OpenBSD ports tree. Between major toolchain updates in base causing some fallout, expanding our Software Defined Radio (SDR) ecosystem, updating my repository of GNUstep app wrappers, and performing long-overdue dependency pruning for some popular web scanners, there is plenty to cover. 

Here is a comprehensive breakdown of the updates, new imports, and maintenance work happened over the last fortnight.

---

## The LLVM 22 Upgrade & GNUstep Fallout

The update of **LLVM to version 22** in OpenBSD base was a fantastic milestone, but as with many major compiler jumps, it introduced a wave of stricter compilation constraints and deprecations. As the maintainer of a significant portion of our **GNUstep** ecosystem, I watched the build infrastructure light up with failures. 

Fixing these required diving into archaic Objective-C runtime behavior, patching underlying code, and collaborating with upstream developers.

### Fixed and Upstreamed
For several ports, the patches I wrote to overcome the LLVM 22 build errors have already been successfully merged into their respective upstreams, or PRs are waiting for review:
* `x11/gnustep/gworkspace` - GNUstep workspace manager
* `x11/gnustep/lapispuzzle` - tetris like puzzle game
* `x11/gnustep/netclasses` - asynchronous networking framework for GNUstep
* `x11/gnustep/examples` - GNUstep example applications

### Local Port Fixes
For others, the fixes were taken from upstream, or not necessarily worth upstreaming:
* `x11/gnustep/base` — GNUstep base library
* `x11/gnustep/gui` — GNUstep gui library
* `x11/gnustep/pantomime` — framework to major mail protocols
* `x11/gnustep/paje` — GNUstep based trace visualization tool
* `x11/gnustep/gmastermind` — GNUstep mastermind game
* `x11/gnustep/renaissance` — GNUstep layer to write portable GUIs

### Rescuing Gomoku
A unique challenge appeared with `x11/gnustep/gomoku`. Upon attempting to patch it, I discovered that the original upstream repository has vanished from the internet. Rather than letting the port die, I coordinated with the maintainer of **GAP (The GNUstep Application Project)**. We successfully imported Gomoku directly into the GAP project, securing its long-term future and upstream home before applying the LLVM 22 fixes.

---

## Added a number of new GNUstep App Wrappers to my Repository

To complement the core ecosystem fixes and ensure non-GNUstep desktop applications fit seamlessly into a clean NEXTSTEP-style workstation experience, I have put together a dedicated collection of **GNUstep App Wrappers**. 

These wrappers let you integrate mainstream graphical applications natively into `GWorkspace` or your GNUstep dock. The complete collection is hosted on [GitHub](https://github.com/buzzdeee/gnustep_apps).

I have freshly added wrapper configurations for the following applications:
* `audacity` — Audio editing suite
* `cubicsdr` — Spectrum visualizer (tying directly into the new SDR ports below!)
* `ghidra` — NSA's reverse engineering framework
* `gimp` — GNU Image Manipulation Program
* `luanti` — The voxel game engine (formerly known as Minetest)
* `prusaslicer` — 3D printing slicer suite
* `rocview` — Rocrail train control system UI
* `spdrs60` — SpdrS60 railway interlocking simulator
* `xclicker` — X11 fast auto-clicker
* `xtrkcad` — CAD program for designing model railroad layouts

---

## Expanding the Software Defined Radio (SDR) Ecosystem

Software Defined Radio on OpenBSD has received a massive shot in the arm. The primary objective of this push was to fully port powerful SDR applications like **[CubicSDR](https://github.com/cjcliffe/CubicSDR)** and **[Gqrx](https://www.gqrx.dk/)**, alongside the key runtime framework needed to support them. 

Central to this ecosystem is **[SoapySDR](https://github.com/pothosware/SoapySDR/wiki)**, which serves as a vendor-neutral hardware abstraction layer. By porting SoapySDR alongside individual hardware modules, OpenBSD now provides a unified API. Any application written against SoapySDR can interact cleanly with varied underlying radio devices without needing specialized individual drivers. For the time being, these are **[RTL-SDR](https://www.rtl-sdr.com/)** and **[Hack RF One](https://greatscottgadgets.com/hackrf/one/)** and **[HackRF Pro](https://greatscottgadgets.com/hackrf/pro/)** devices.

### New Imports
* `comms/cubicsdr`: A beautiful cross-platform Software-Defined Radio application built using wxWidgets.
* `comms/soapysdr`: The vendor-neutral SDR data abstraction library.
* `comms/soapy-rtlsdr`: Wrapper to bring RTL-SDR dongle support to SoapySDR.
* `comms/soapy-hackrf`: Wrapper enabling HackRF hardware under the Soapy framework.

### Updates and Ownership Changes
* `comms/hackrf`: Updated to support latest firmware and runtime requirements, most notably adding support for **HackRF Pro**.
* `comms/dump1090`: Updated the popular ADS-B aircraft tracker. While under the hood, I officially stepped up and **took MAINTAINER** for this port to ensure its continuous care.

### Awaiting Review at `ports@`
To complete the SDR ecosystem overhaul, I have submitted two crucial frontend additions to the mailing list for review:
* `comms/gqrx`: A powerful AM/FM/SSB software-defined radio receiver graphical user interface.
* `comms/gr-osmosdr`: The GNU Radio block for various radio hardware.
* `comms/sdrpp`: bloat-free SDR software with a focus on performance

If you are running an SDR setup on OpenBSD, keep an eye out for these or grab the Makefiles from the list to help test!

Related to the topic of SDR, see my previous article [Fun with RTL-SDR on OpenBSD: Planes, Meters, and Radio](/en/fun-with-rtl-sdr/)

{{< admonition type=note title="Note on Hardware Alternatives" open=true >}}
If HackRF devices are a bit too pricey for your budget: The well-known **RTL-SDR Blog V3/V4** dongles are also available on eBay at a great price. You can find matching antennas there as well... Just use this [affiliate link to eBay](https://www.ebay.de/sch/i.html?_nkw=RTL-SDR+Blog&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=RTLSDR&toolid=10001&mkevt=1).
{{< /admonition >}}

---

## Web Scanners & Ruby Dependency Housekeeping

Updating security scanners is often a game of dominoes. Bringing **wpscan** and **whatweb** up to speed allowed me to do a massive sweep of the Ruby ports tree, deleting obsolete libraries and modernizing others.

### WPScan Revamp & Pruning
The `security/wpscan` update required a shift in how it handles headless browser interaction. This allowed me to drop a legacy framework and adopt a cleaner dependency chain:

* `www/ruby-ferrum`: **NEW** port added to provide high-level Chrome/Chromium automation.
* `security/ruby-cms_scanner`: **DELETED** (only used by the old wpscan version).
* `devel/ruby-opt_parse_validator`: **DELETED** (orphaned after `ruby-cms_scanner` was removed).

### Cascading Ruby Dependency Updates
To satisfy the newer scanner environments, a fleet of other Ruby libraries were bumped:
* `devel/ruby-activesupport` — Updated
* `net/ruby-public_suffix` — Updated
* `net/ruby-connection_pool` — Updated
* `www/ruby-addressable` — Updated
* `www/ruby-xmlrpc` — Updated

### Taking Over WhatWeb
* `net/whatweb`: Updated the next-generation web scanner to its latest version. While digging through the updates, I also **took MAINTAINER** here to keep its development alignment tight on OpenBSD.

---

## Miscellaneous Updates & A Treat for the Kids

To round out the last two weeks, several independent components across the tree received attention:

### Infrastructure & Application Bumps
* `net/ruby-msgpack` & `www/ruby-faraday-net_http`: Updated (both are critical dependencies for `openvox`).
* `textproc/py-commonmark`: Updated (dependency for `puppetboard`).
* `archivers/ruby-rubyzip`: Updated to the latest stable release.
* `audio/qsynth`: Updated the fluidsynth GUI front-end.
* `security/exploitdb`: Updated to ensure local exploit payload definitions match current public archives.

### Just For Fun: XClicker
Finally, I submitted a new port to `ports@` for review:
* `x11/xclicker`: A fast, feature-rich graphical auto-clicker for X11.

*Why?* Well, kids play silly clicker games on their computers these days and frequently complain about their fingers hurting. Now they can cheat efficiently and natively on OpenBSD. 😉

---

## Wrapping Up

Two weeks, dozens of packages built, toolchain breakage neutralized, a robust new SDR stack ready for action, and wrappers configured to build a tighter desktop workflow. 

If you are running `-snapshots`, simply update and try the new SDR additions. If you're running GNUstep based desktop, check out the app wrappers repository to round out your deployment, and as always, feedback or testing for the ports awaiting review on `ports@` is highly encouraged. 

