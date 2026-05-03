---
title: "Tracks and Daemons: Digital Model Railroading on OpenBSD"
subtitle: "Resurrecting DCC hardware and fighting bitrot in the Ports tree"
date: 2026-05-02T17:19:11+02:00
lastmod: 2026-05-02T17:19:11+02:00
draft: false
description: "A journey from analog childhood trains to a fully digitized layout controlled by OpenBSD."
license: "MIT"
images: [ "/images/model_railroad_openbsd.png", "/images/rocview_main.png", "/images/Rocrail_OpenDCC_Z1_options.png", "/images/spdrs60.png", "/images/dtcltiny.png", "/images/xtrkcad.png" ]

tags: ["OpenBSD", "ModelRailroad", "DCC", "Rocrail", "srcpd", "XTrackCAD", "C++", "Electronics"]
categories: ["Tech", "Hacking"]

featuredImage: "/images/model_railroad_openbsd.png"
featuredImagePreview: "/images/model_railroad_openbsd.png"

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
  images: [ "/images/model_railroad_openbsd.png", "/images/rocview_main.png", "/images/Rocrail_OpenDCC_Z1_options.png", "/images/spdrs60.png", "/images/dtcltiny.png", "/images/xtrkcad.png" ]
  # ...
---

## The World in Miniature

Like many kids of my generation, I spent countless hours in the world of analog model railroading. But for me, it wasn't just about watching a locomotive circle a loop of track. The real magic was in the **creation**: assembling plastic house kits, meticulously crafting terrain, and the complex "under-the-table" work of wiring up lights and electromagnetic turnouts. It was my first introduction to engineering and project management, though I didn't know it then.

Eventually, the layout was dismantled and boxed up, staying dormant for years as my focus shifted from physical circuits to digital ones—specifically, the world of Unix and OpenBSD.

## The 2009 Revival: The DIY Command Station

Around 2009/10, I finally pulled those boxes out of storage. Looking at the old analog transformers, I knew I wanted to modernize. I wanted **[DCC (Digital Command Control)](https://en.wikipedia.org/wiki/Digital_Command_Control)**, but I wanted to keep that "hands-on" spirit I had as a kid.

The world of Digital Command Control (DCC) is fascinating, but mostly dominated by proprietary, "black box" hardware. Being an OpenBSD user,
I naturally gravitated toward something more open. At that time I discovered the **[OpenDCC Z1](https://www.opendcc.de/elektronik/opendcc/opendcc.html)** station.

The best part? You could buy the PCB and solder the components yourself. There is something incredibly satisfying about building your own command station from the ground up. I spent weeks hunched over the PCB with a fine-tip iron and a magnifying glass. There is an incredible, nerve-wracking satisfaction in soldering for weeks on end, not knowing if the board will actually work or if you've accidentally fried a critical microcontroller with a stray bridge. When that first LED finally blinked and the board responded to a ping, the relief was immense.

{{< figure
    src="/images/OpenDCC_Z1.jpeg"
    alt="OpenDCC Z1"
    caption="OpenDCC Z1"
    class="ma0 w-75"
>}}

However, hardware is only half the battle. I needed a way to talk to the Z1 from my favorite operating system.

The hardware was alive, but the "OpenBSD factor" remained. In 2010, the landscape for model railroading on OpenBSD was a bit of a desert. To run a layout from my OS of choice, I needed a software stack. I got to work porting and packaging the essentials for the OpenBSD ports tree:

* **[srcpd](https://sourceforge.net/projects/srcpd/)**: The daemon acting as the SRCP protocol gateway.
* **[spdrs60](https://sourceforge.net/projects/spdrs60/) & [dtcltiny](https://sourceforge.net/projects/dtcltiny/)**: Traditional GUI control panels.
* **[XtrkCad](https://sourceforge.net/projects/xtrkcad-fork/)**: For the precision planning of track geometries.
* **[Rocrail](https://www.rocrail.online/)**: The comprehensive automation suite.

I got everything working, the ports were committed, and I was ready to build the layout of my dreams. But then... life happened. Kids arrived, free time evaporated, and the railroad was once again boxed up for more than a decade.

## The Resurrection (and the Bitrot)

Late last year, my kids started asking: *"Hey Dad, what's in those big boxes in that corner?"* It was time.

We pulled everything out and set up a temporary layout. But 10 years is a lifetime in software development. When I tried to update my old OpenBSD ports, I hit a wall of bitrot.

### The Rocrail Challenge

I discovered that **Rocrail** had transitioned to a closed-source model years ago. For an OpenBSD user, this is usually a dead end. Furthermore, the version in our ports tree was marked as broken some time ago as well; it simply wouldn't build because modern `wxWidgets` versions had moved on, leaving the old code behind.

But I remembered how well Rocrail worked in the past, and I wasn't ready to give up on it. I found a fork on GitHub that preserved the last open-source state of the project and dug in. I spent several late nights hacking on the C++ code, fixing `wxWidgets` incompatibilities, squashing compilation warnings, and wrestling the build system until it finally played nice with current OpenBSD. There’s a specific kind of triumph in seeing a `clean build` message after a decade of neglect. For those curious, might want to take a look at the recent commits in my [Rocrail GitHub](https://github.com/buzzdeee/rocrail) repo.

## Current Status: Green Lights and Minor Glitches

After the "Great Fixing," I tested the whole stack again: `srcpd`, `spdrs60`, `dtcltiny`, and `Rocrail`. 

**The Good:**
* **XtrkCad:** Which I kept up-to-date over the years, still works like a charm. It remains a rock-solid tool for layout design.

{{< figure
    src="/images/xtrkcad.png"
    alt="XTrackCad on OpenBSD"
    caption="XTrackCad on OpenBSD"
    class="ma0 w-75"
>}}

* **Rocrail:** It’s back from the dead! It is working solidly. Interestingly, I can enable the internal `srcpd` service within Rocrail and connect the older tools like `spdrs60` and `dtcltiny` to it. It’s a very flexible setup.

{{< figure
    src="/images/rocview_main.png"
    alt="Rocrails Rocview on OpenBSD"
    caption="Rocrails Rocview on OpenBSD"
    class="ma0 w-75"
>}}

{{< figure
    src="/images/Rocrail_OpenDCC_Z1_options.png"
    alt="OpenDCC Z1 setup in Rocrail"
    caption="OpenDCC Z1 setup in Rocrail"
    class="ma0 w-75"
>}}

* **spdrs60 & dtcltiny:** My testing shows these are reasonably stable when connected against the Rocrail `srcp` service.

{{< figure
    src="/images/spdrs60.png"
    alt="Spdrs60 on OpenBSD"
    caption="Spdrs60 on OpenBSD"
    class="ma0 w-75"
>}}

{{< figure
    src="/images/dtcltiny.png"
    alt="Dtcltiny on OpenBSD"
    caption="Dtcltiny on OpenBSD"
    class="ma0 w-75"
>}}

**The Not-so-Good:**
* **srcpd** seems to have developed a bit of a temper. It occasionally struggles to maintain a stable connection with my OpenDCC Z1, sometimes leaving the hardware in a locked state that requires a power cycle. It’s a bug I'll need to hunt down in the serial communication code.

{{< admonition type=info title="What is srcpd?" open=true >}}
The **srcpd** ([Simple Railroad Command Protocol](https://srcpd.sourceforge.net/srcp/) daemon) acts as a universal interface between software and hardware. It abstracts the complexity of various DCC protocols and provides a standardized network server.

**How clients connect:**
* **Protocol:** Communication happens over a simple, text-based TCP protocol (defaulting to port **4303**).
* **Flexibility:** Since the protocol is open and well-documented, multiple clients can connect simultaneously. While **Rocrail** handles automation, tools like **spdrs60** (acting as a virtual signal box) or **dtcltiny** (as a throttle) can access the same hardware in parallel.
* **Modularity:** On OpenBSD, this enables a clean, modular setup where the daemon exclusively manages the serial interface to the Z1, while the GUI programs act as lightweight network clients.
{{< /admonition >}}

## What's Next?

Even with the quirks, it is incredibly rewarding to see a locomotive move across the track because of a command sent from an OpenBSD machine using code I helped maintain.

The mission isn't over. While re-enabling Rocrail didn't make it into the upcoming OpenBSD 7.9 release, I have just today sent an email to `ports@` for a review of the updated port. My hope is to get it officially unbroken and re-enabled once the release unlock occurs.

I’ve also just bought some **[S88 feedback modules](https://shop.dcc-versand.de/bausatz-as88-gleisbesetztmelder-stromsensor-p-1872.html)** to add to the Z1. S88 is definitely "old-school" tech, but it is well supported by the Z1 and this version of Rocrail. It allows for basic occupancy detection—knowing a section of track is blocked. 

While the software seemingly has some support for **BiDi (Bi-Directional communication)**, which would allow the system to identify *which* specific train is on a block, that requires decoders that support it (which I don't have yet!). For now, the goal is simple: get the old-school S88 working reliably on the Z1.

Beyond Rocrail, I'm also looking into **[JMRI](https://www.jmri.org/)**. It's a powerful tool, especially for decoder programming. However, I'm a bit wary of the serial port access under Java on OpenBSD—an issue I recently encountered while looking into **[openHAB](https://www.openhab.org/)**, where `rxtx` failed to work entirely. Whether `purejavacomm` can save the day here remains to be seen.

Just like when I was a kid, the project is never truly "finished"—and that’s exactly the point. Expect more model railroad hacking updates soon!

