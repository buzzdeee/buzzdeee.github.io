---
title: "Fujitsu Futro S920 and OpenBSD"
subtitle: "Hands-on Report on Hardware, Compatibility, and Use Cases"
date: 2026-09-02T22:46:03+02:00
lastmod: 2026-09-02T22:46:03+02:00
draft: false
description: "A practical summary of using the Fujitsu Futro S920 Thin Client with OpenBSD over several years."
license: "MIT"
images: [ "/images/Fujitsu_Futro_S920_featured.jpg", "/images/futro-s920.jpg", "/images/futro-s920-open.jpg" ]

tags: ["OpenBSD", "hardware", "thinclient", "futro", "server", "homelab"]
categories: ["Hardware", "OpenBSD"]

featuredImage: "/images/Fujitsu_Futro_S920_featured.jpg"
featuredImagePreview: "/images/Fujitsu_Futro_S920_featured.jpg"

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
mapbox:
share:
  enable: true
comment:
  enable: true
library:
  css:
  js:
seo:
  images: [ "/images/Fujitsu_Futro_S920_featured.jpg", "/images/futro-s920.jpg", "/images/futro-s920-open.jpg" ]
---

Over the course of a few years, several Fujitsu Futro S920 units served as reliable, silent servers for various local network services in my homelab. I have since gradually upgraded the infrastructure and replaced the old instances with newer Futro S9010 models.

Before these decommissioned S920 devices find a new home on [eBay](https://ebay.us/nn1zUq), here is a practical overview covering their hardware, power consumption, and OpenBSD compatibility. Even though the architecture isn't cutting-edge anymore, these thin clients still make fantastic, power-efficient nodes.

<figure class="ma0 w-75">
  <img src="/images/futro-s920.jpg" alt="Fujitsu Futro S920 Front and Rear View">
  <figcaption>The Fujitsu Futro S920 in front and rear view.</figcaption>
</figure>

---

## Hardware Features and Interfaces

The Futro S920 relies on a completely fanless, passively cooled design without any moving parts. The chassis is easy to open, making maintenance or swapping out components a breeze.

{{< admonition type="info" title="Hardware Overview & Expansion" >}}
* **Processor:** AMD GX-222GC SOC (2 cores, 2.2 GHz, Turbo up to 2.4 GHz) with integrated Radeon R5E graphics
* **Memory (RAM):** 2x DDR3 SO-DIMM slots. The official Fujitsu datasheet states support for up to 8 GB. In practice, depending on the module configuration and BIOS version, up to **16 GB** (2x 8 GB SO-DIMM) can often be addressed.
* **M.2 / mSATA:** 1x mSATA slot (ideal for a compact system SSD to boot the OS)
* **SATA Port:** 1x native 2.5″ SATA port including an onboard power header (for larger SSDs or HDDs for data storage)
* **Expandability:** 1x PCIe slot (requires an internal riser card)
* **Serial Ports:** 2x native RS-232 interfaces (`com0` and `com1`) on the rear
* **Networking:** 1x Gigabit Ethernet (Realtek RTL8168/8111G)
* **Power Supply & Consumption:** Powered by standard external 19V laptop power bricks. Power draw under OpenBSD idling is a frugal **7 to 9 Watts**. Under full synthetic load, consumption rarely climbs past **15 to 18 Watts**.
* **Specs & Manual:** [Fujitsu FUTRO S920 Operating Manual](https://support.ts.fujitsu.com/Search/SWP1206702.asp)
{{< /admonition >}}

The separation between the mSATA slot (perfect for the OpenBSD OS installation) and the 2.5″ SATA port (for storage) makes the platform surprisingly versatile for storage or network tasks.

<figure class="ma0 w-75">
  <img src="/images/futro-s920-open.jpg" alt="Fujitsu Futro S920 Internal View">
  <figcaption>Inside the Fujitsu Futro S920.</figcaption>
</figure>

---

## Hardware Detection under OpenBSD

All onboard integrated chipsets are supported natively by the OpenBSD kernel out of the box. No extra gymnastics are required for network hardware, graphics, or audio.

The following snippet shows the original boot log (`dmesg`) under OpenBSD 8.0-beta:

{{< highlight text >}}
{{< /highlight >}}

{{< admonition type="note" title="Graphics & Xorg Setup" >}}
The `radeondrm0` driver attaches the graphics hardware ("AMD Mullins") natively. Both the terminal console (`wsdisplay0`) in full resolution (1920x1080) and an optional X11 server run smoothly without needing manual driver tweaks.
{{< /admonition >}}

{{< admonition type="warning" title="Important Note: Fixing DRM Warnings in dmesg" >}}
If DRM warning messages appear in `dmesg` during system boot, such as:

{{< highlight text >}}
drm:pid0:drm_fb_helper_find_format *WARNING* [drm] format 0x36314752 not supported
drm:pid0:drm_fb_helper_find_format *WARNING* [drm] format 0x36314752 not supported
drm:pid0:__drm_fb_helper_find_sizes *WARNING* [drm] No compatible format found
{{< /highlight >}}

the culprit is usually an undersized video framebuffer allocated in the system BIOS.

**Fix:** Enter the BIOS during boot and increase the **UMA Framebuffer Memory to at least 256MB**. After that, `radeondrm0` initializes cleanly, and the system runs at full display resolution without errors.
{{< /admonition >}}

---

## Past Use Cases

Given its specification and low power draw, the S920 handled the following tasks flawlessly in my setup:

1. **Infrastructure Services:** Running `nsd` alongside `unbound` (local DNS resolver), `dhcpd`, and serving PXE/TFTP boot images via `tftpd` for fresh OS installs on the network.
2. **Serial Console Host:** Managing external headless equipment via the two rear serial ports (`com0`/`com1`).
3. **Kiosk / Light Workstation:** Occasional use with a graphical interface for lightweight monitoring dashboards.

---

## Summary & Outlook

Moving to the Futro S9010 brings extra computing performance, modern CPU instructions, and an updated platform. Nevertheless, the Futro S920 remains a remarkably capable, completely silent piece of hardware for lightweight setups, OpenBSD beginners, or simple home network servers.

If you are looking for an inexpensive used machine with native serial ports and low power draw, a quick search on [eBay](https://ebay.us/UlXc9q) yields plenty of options with various configurations. The S920 provides solid hardware value for very little money.

