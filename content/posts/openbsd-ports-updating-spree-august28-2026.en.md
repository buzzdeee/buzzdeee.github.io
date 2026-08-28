---
title: "Fresh Air in OpenBSD Ports: SDR, Pentesting Tools, OpenVOX, and Plaso"
subtitle: "A look back at recent ports updates, frequency switching, and forensics hurdles"
description: "A detailed look at the latest OpenBSD ports updates: From SDR imports like Gqrx and ReadsB to security tools and the tricky Plaso and Libyal upgrade."
date: 2026-08-28T22:00:00+02:00
draft: false
license: "MIT"
tags: ["OpenBSD", "Ports", "SDR", "Gqrx", "Plaso", "OpenVOX", "Security"]
categories: ["OpenBSD", "Tech", "Porting"]
images: [ "/images/ports-sprint-august-2026.jpg", "/images/Gqrx.png" ]

featuredImage: "/images/ports-sprint-august-2026.jpg"
featuredImagePreview: "/images/ports-sprint-august-2026.jpg"

toc:
  enable: true
  auto: true
  keepStatic: false
code:
  copy: true
  maxShownLines: 50
math:
  enable: false
share:
  enable: true
comment:
  enable: true
seo:
  images: [ "/images/ports-sprint-august-2026.jpg", "/images/Gqrx.png" ]
---

It was about time: The [last major upgrade sprint](/en/openbsd-ports-updating-spree-june4/) in the OpenBSD ports tree was quite a while ago, and a backlog of packages needing some serious TLC had accumulated on my to-do list. Anyone familiar with the OpenBSD ports ecosystem knows that such an update rarely consists of merely bumping a version number in a `Makefile`. Instead, it is a lively journey of discovery through dependency trees, build and test changes, compilation errors, and more.

Over the past few days and weeks, I worked my way through several distinct areas: ranging from Software Defined Radio (SDR) to pentesting and web security tools, all the way to infrastructure automation with OpenVOX and digital forensics centered around Plaso. Here is an overview of the work and the stories behind the commits.

---

## Frequencies, Planes, and Frequency Switching: The SDR Domain

The SDR section received a substantial upgrade. I was particularly thrilled about importing **`comms/gqrx`** along with its required dependency **`comms/gr-osmosdr`**. Gqrx is an extremely popular, open-source SDR receiver built on top of GNU Radio and the Qt GUI toolkit. This provides another excellent graphical interface on OpenBSD for exploring the radio spectrum.

{{< figure
    src="/images/Gqrx.png"
    alt="Gqrx in action"
    caption="Gqrx in action"
    class="ma0 w-75"
>}}

An important tip came from `jbg@`, who pointed me toward **`comms/readsb`**. `readsb` is a high-performance Mode-S/ADS-B/TIS decoder for hardware like the RTL-SDR, HackRF, or Mode-S Beast.

However, `readsb` brought quite a few build issues along for the ride, having been written and tested more or less exclusively for Linux. It required numerous patches across various parts of the codebase to get it building and running cleanly on OpenBSD. Overcoming those incompatibilities definitely took a bit of time—but the effort was more than worth it!

{{< admonition type="tip" title="Practical Tip: Combining ReadsB & Dump1090" open=true >}}
In my testing, `readsb` reliably picked up more aircraft—at least when using an RTL-SDR Blog dongle—than the existing `dump1090` in ports. The only catch: `readsb` does not come with its own web interface for map display out of the box. The solution, however, is quite elegant:
1. Start `dump1090` in **network-only mode**.
2. Configure `readsb` to forward aircraft data via the **Beast protocol** to `dump1090`.

Result: The superior decoding performance of `readsb` paired with the neat map visualization of `dump1090`!
{{< /admonition >}}

For anyone looking to dive into the world of signals: Reliable [RTL-SDR Blogs are available on eBay](https://ebay.us/XTunjd#affiliate) as an ideal hardware foundation.

### Stability Under the Hood

Beyond new imports, there were also crucial stability fixes. **SASANO Takayoshi (`uaa@`)** contributed important corrections in `comms/rtl-sdr` and `comms/soapy-rtlsdr`: Switching from asynchronous to synchronous calls ensures that frequency switching now works smoothly without annoying hangs or crashes.

**Overview of SDR Updates:**
* `comms/gr-osmosdr` *(Newly imported)*
* `comms/gqrx` *(Newly imported)*
* `comms/readsb` *(Newly imported)*
* `comms/liquid-dsp`: Update 1.7.0 -> 1.8.2
* `comms/rtl-sdr` & `comms/soapy-rtlsdr` *(Stability fixes)*

Read more about RTL-SDR in my previous blog post [here](/en/fun-with-rtl-sdr/).

---

## Development & Pentesting Tools

Next up was the area of security and development tooling. The popular reverse-engineering tool `devel/apktool` was bumped to version 3.0.3, `security/exploitdb` received a fresh dataset, and `security/py-fickling` got a minor update as well.

Updating the WordPress security scanner `wpscan` (to version 4.1.0) also allowed updating a whole stack of its dependencies:

{{< highlight yaml >}}
security/wpscan:                4.0.0     -> 4.1.0
devel/ruby-activesupport:       8.1.3     -> 8.1.3.1
www/ruby-ethon:                 0.16.0    -> 0.18.0
www/ruby-typhoeus:              1.4.1     -> 1.6.0
www/ruby-ferrum:                0.17.2    -> 0.18.0
security/py-fickling:           0.1.11    -> 0.1.12
security/exploitdb:             2026-06-02 -> 2026-08-19
{{< /highlight >}}

While already working in that corner, it was a good opportunity to tidy up side targets: Even though no direct internal dependencies existed, `archivers/ruby-rubyzip` (to 3.5.0) and `devel/ruby-zeitwerk` (to 2.8.3) were updated along the way.

---

## Infrastructure: OpenVOX & Puppetboard

There was also movement in the OpenVOX (formerly known as Puppet) ecosystem. Alongside various Ruby libraries, the server component `sysutils/openvox-server/8` was brought from version 8.14.0 to 8.15.2. For the `puppetboard` monitoring frontend, its dependencies `py-cachelib` (0.17.0) and `py-flask-caching` (2.5.0) were updated as well.

* `net/ruby-msgpack`: 1.8.1 -> 1.8.4
* `www/ruby-faraday`: 2.14.2 -> 2.14.3
* `sysutils/ruby-openvoxserver-ca`: 3.2.0 -> 3.3.0
* `sysutils/openvox-server/8`: 8.14.0 -> 8.15.2

---

## Forensics with Plaso & the Libyal Rollercoaster

The most time-consuming part of this sprint was updating the digital forensics toolchain surrounding **Plaso** (to version 20260720; see also the official [Plaso Release Notes](https://osdfir.blogspot.com/2026/07/plaso-20260720-released.html)).

Plaso brings along a massive web of `libyal` libraries. I took this opportunity to bring order to the ports tree by consistently moving `sysutils/libfsntfs` over to `sysutils/libyal/libfsntfs`.

{{< admonition type="note" title="Build Roadblocks: Autoconf & FUSE" open=true >}}
While building the updated `libyal` ports, I ran straight into two interesting issues:

1. **Autoconf Confusion:** The `libyal` suites switched their test generation over to `autoconf`. After initial build failures and a brief but very helpful exchange with upstream maintainer **Joachim Metz** on GitHub, it turned out that I was simply using the wrong Autoconf version. Once switched to the correct version, the process ran smoothly like clockwork.
2. **FUSE Incompatibility:** All `libyal` filesystem ports (`libfsfat`, `libfsxfs`, etc.) failed on the `fuse_unmount()` function. The background: OpenBSD uses an older FUSE 2.X interface in its base system, whereas `libyal` assumed FUSE 3.X API behavior. Creating appropriate patches resolved the issue. The upstream maintainer has been informed and will adapt all `libfsXXX` packages accordingly.
{{< /admonition >}}

Thanks to the excellent and extensive test suites provided by Plaso and the `libyal` libraries—which finally completed successfully once the proper Autoconf version was in place—the final testing phase was extremely reassuring. Watching hundreds of tests pass successfully through the terminal lets you sleep much better as a porter.

---

## Summary and Outlook

As it turned out, quite a bit had accumulated since the last sprint. It feels great to know that, relatively close to the OpenBSD 8.0 release freeze, the vast majority of the ports I maintain are up to date.

Hopefully, only minor updates will be needed here and there between now and the freeze—keeping things completely stress-free right before the release! Until then: Have fun testing the new ports, and Happy Hacking!

