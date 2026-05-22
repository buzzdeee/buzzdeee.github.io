---
title: "OpenBSD Ports Updating Spree: OpenVox, DFIR Tools, and Fixing Rocrail"
subtitle: "Tackling the never-ending backlog of package updates."
date: 2026-05-22T19:45:44+02:00
lastmod: 2026-05-22T19:45:44+02:00
draft: false
description: "A look into this week's OpenBSD ports updating marathon, featuring major dependency overhauls for OpenVox, Volatility3, Kismet, and getting model railroads back on track."
license: "MIT"
images: [ "/images/ports-updating-sprint.png" ]

tags: ["OpenBSD", "Ports", "Security", "Ruby", "Puppet", "Rocrail"]
categories: ["Tech"]

featuredImage: "/images/ports-updating-sprint.png"
featuredImagePreview: "/images/ports-updating-sprint.png"

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
  images: [ "/images/ports-updating-sprint.png" ]
  # ...
---

I spent a chunk of time this week on a focused OpenBSD ports updating sprint, putting a number of other projects a bit to the side to finally tackle this overdue activity. As it usually goes, what started as a few simple bumps quickly turned into a giant domino effect of interconnected dependencies—especially in the Ruby and Python ecosystems ;)

It's not worth listing every single minor bump explicitly, but here is a breakdown of the major updates that just hit the tree.

## OpenVox & Puppet Ecosystem

A lot of work went into the `sysutils/openvox-server` architecture and its surroundings this round. 

* **[openvox-server](https://voxpupuli.org/openvox/)** updated to 8.13.0, along with **[r10k](https://github.com/puppetlabs/r10k)** (5.0.3) and **[puppetboard](https://github.com/voxpupuli/puppetboard)** (7.0.2).
* **[ruby-pdk](https://github.com/puppetlabs/pdk)** and its dependencies required a bunch of work. I updated a fair number of them and relaxed their version requirements. Since the meanwhile not soo rechent anymore move from Puppet to OpenVOX, parts of PDK are broken, but it’s still good for quickly spinning up a bare-bones module or class to help you get going.

## Network & Security / Forensics

The digital forensics (DFIR) and wireless security stacks got a heavy refresh. 

* **[volatility3](https://github.com/volatilityfoundation/volatility3)** (2.28.0), **[pdf-parser](https://blog.didierstevens.com/my-software/)** (0.7.14), **[py-fickling](https://github.com/trailofbits/fickling) **, **[apktool](https://apktool.org/)**, and **[exploitdb](https://gitlab.com/exploit-database/exploitdb)** are all up to date.
* **[kismet](https://www.kismetwireless.net/)**: This got some nice improvements. It now links against `libpcap` in ports, which allows converting `kismetdb` captures to `pcap` for BTLE and NRF captures. I also got the [Zigbee: TICC 2531] support working, allowing to use [TI CC2531 Zigbee](https://www.ebay.de/itm/404481824304?var=674447321503&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=TICC2531&toolid=10001&mkevt=1) to survey Zigbee based wireless network. If you want more details on that, I covered the setup in an older post: [Zigbee Sniffing on OpenBSD: Diving Deeper with the TI CC2531](/en/zigbee-sniffing-on-openbsd/).
* **[plaso](https://github.com/log2timeline/plaso)** & **[sleuthkit](https://sleuthkit.org/sleuthkit/)**: Upgraded Sleuthkit to 4.15.0 and Plaso to 20260512. This required updating the whole underlying Python dependency chain (`py-acstore`, `py-dtfabric`, `py-dfvfs`, etc.). 
* While I was working on `sysutils/py-tsk`, I realized I forgot to update it back in February, so I went ahead and officially took over as the **MAINTAINER** for it.

## Model Railroad, Audio, and Games

Ports work isn't all server automation and security; From time to time, or the kids, need a break ;) So I also took care of some entertainment packages.

* **[rocrail](https://www.rocrail.online/doku.php?id=start)**: This was actually broken for some years, but I got it updated to build and run again, along with a handful of other under-the-hood improvements. I wrote about the endeavour in a previous article: [Tracks and Daemons: Digital Model Railroading on OpenBSD](/en/digital-model-railroading-on-openbsd/).
* **[qsynth](https://qsynth.sourceforge.io/qsynth-index.html)**: Upgraded from 1.0.3 to 1.0.5.
* **games**: Minor version bumps for both **[emptyclip](https://empty-clip.gitlab.io/)** and **[choria](https://choria.gitlab.io/)**.

## While I was there...

I also cleaned up a few miscellaneous things along the way:
* Bumped the Java dependency for **devel/jenkins** up to Java 21.
* relax dependencies on ruby-fast_gettext in **sysutils/ruby-openvox/8**


## Wrapping up

With this sprint wrapped up, the backlog of outdated ports has been reduced dramatically. It’s still not down to zero—though let's be honest, it probably never will be ;) Time will inevitably move on and "magically" make that list grow again all on its own. 

Until then, everything is available in snapshots now, committed, and ready to go. Run your updates and let me know if anything breaks!


