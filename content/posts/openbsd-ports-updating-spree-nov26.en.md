---
title: "OpenBSD Ports Sprint: From OpenVox Security Fixes to Kismet & Plaso"
subtitle: "Another sweep through the OpenBSD ports tree featuring critical security updates, Ruby gems, and forensics tools"
date: 2026-11-23T21:10:28+01:00
lastmod: 2026-11-23T21:10:28+01:00
draft: false
description: "Summary of an extensive OpenBSD ports update sprint covering OpenVox, WPScan, Plaso, Kismet, GNUstep, and openHAB."
license: "MIT"
images:
  - "/images/openbsd-ports-update-spree-nov26.jpg"
  - "/images/puppetboard-in-action.png"
  - "/images/kismet_in_action.png"

tags:
  - OpenBSD
  - Ports
  - OpenVox
  - Security
  - WPScan
  - Plaso
  - Kismet
  - Ruby
  - SDR
categories:
  - OpenBSD
  - Porting

featuredImage: "/images/openbsd-ports-update-spree-nov26.jpg"
featuredImagePreview: "/images/openbsd-ports-update-spree-nov26.jpg"

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
    - "/images/openbsd-ports-update-spree-nov26.jpg"
    - "/images/puppetboard-in-action.png"
    - "/images/kismet_in_action.png"
---

## It Was About Time …

... for an extensive OpenBSD ports update sprint! A regular glance at my [Portroach overview](https://portroach.openbsd.org/sebastian%20reitenbach%20%3Csebastia@openbsd.org%3E.html) reliably reminds me when the list of outdated packages has grown a bit too long and it's high time for the next round. As is so often the case, you start off innocently enough with a couple of individual package updates—and before you know it, you're pulling a whole cascade of dependencies, security advisories, and upstream patches along with you. It is precisely this mix of detective work, dependency juggling, and the satisfaction of keeping the ports tree up to date and secure that makes it so rewarding.

In this round, the focus was heavily on the **OpenVox** ecosystem (the community fork of Puppet), closely followed by a range of exciting **security and forensics tools** like Plaso, Kismet, and WPScan, alongside a few classics from the GNUstep and smart home corners.

---

## The OpenVox and Puppet Ecosystem

**[OpenVox](https://docs.openvoxproject.org/openvox/8.x/)** is the open-source community fork of Puppet, used for declarative configuration management and automation of server infrastructures. Together with **OpenVoxDB** (for storing inventory and report data) and **Puppetboard** (the web frontend for visualization), it forms the backbone of many automated setups.

### Critical Security Updates & Chain Reactions

A few weeks ago, Klemens (`kn@`) pointed out critical security vulnerabilities in `openvoxdb` and `openvox-server` to me. The culprit was a flaw in `trapperkeeper-authorization` ([GHSA-q4hm-33rc-572g](https://github.com/OpenVoxProject/trapperkeeper-authorization/security/advisories/GHSA-q4hm-33rc-572g)) boasting a **CVSS 3.1 score of 9.5**.

{{< admonition type=danger title="Critical Security Vulnerability (CVSS 9.5 & 7.7)" >}}
Due to the vulnerability in `trapperkeeper-authorization` (CVSS 9.5) as well as another flaw discovered shortly thereafter directly in OpenVoxDB ([GHSA-859m-7qpm-85p7](https://github.com/OpenVoxProject/openvoxdb/security/advisories/GHSA-859m-7qpm-85p7), CVSS 7.7), a timely update of all server components was urgently required.
{{< /admonition >}}

The following updates and fixes made their way into the ports tree as part of this effort:

* **openvoxdb**: `8.12.1` -> `8.18.0` -> `8.18.1`
* **openvox-server**: `8.15.2` -> `8.18.0`
* **openvox**: The maintainer `kn@` had already updated the package to `8.32.0`. When I subsequently wanted to update to the latest version `8.34.0`, there was a minor surprise during the dependency check: in `8.32.0`, the Ruby gem's `.metadata` still happily allowed `puppet-resource_api` versions greater than 2. Starting with version `8.33.0`, however, upstream restricted this requirement back to `>= 1.9.2, < 2` (see [https://github.com/OpenVoxProject/openvox/pull/733](https://github.com/OpenVoxProject/openvox/pull/733)). Since I had already updated `devel/ruby-puppet-resource_api` to `2.1.0` in the meantime, I patched the `.metadata` accordingly during the update to `8.34.0`. The background for this upstream restriction likely lies in the all-in-one bundles for other operating systems, where all components are packaged and shipped together. Still waiting on feedback from the maintainer to get the `8.34.0` version into the ports tree.
* **openfact & openvox fixes**: In `openfact`, the network resolver for `gif` and `gre` tunnels has a fix in upstream PR in the queue [here](https://github.com/OpenVoxProject/openfact/pull/256), while in `openvox`, the handling of package branches containing dots (such as `8.4`) was resolved.

### A Detour to the Alpha Architecture

Out of pure curiosity, I also dusted off my old **Alpha workstation** in between to see how the OpenVox client fares on this architecture. To get `openvox` to build, install, and run cleanly there, `sysutils/ruby-augeas` and `devel/ruby-ffi` each needed a small patch in their `Makefile`: previously, `c++abi` was unconditionally listed in `WANTLIB`, whereas it should of course only be included on architectures using **Clang** as the compiler—and not on GCC platforms like `alpha`. Building all the dependencies only took a couple of days, and afterwards I had an Alpha client connected to my OpenVox server. The practicality of this endeavor is not up for debate here ;)

### Ruby Gems, PDK, r10k, and Puppetboard

To ensure OpenVox, the Puppet Development Kit (PDK), `r10k` (the tool for Git-based environment and module management), and `hiera-eyaml-gpg` for encrypted Hiera data play nicely together, there was a whole series of gem updates—including the **Puppetboard** web dashboard:

* **Puppet & PDK dependencies**:
  * `archivers/ruby-minitar`: `1.1.0` -> `1.2.0`
  * `net/ruby-msgpack`: `1.8.5` -> `1.8.7`
  * `devel/ruby-puppet-resource_api`: `2.0.1` -> `2.1.0`
  * `devel/ruby-fast_gettext`: `4.1.1` -> `4.2.0`
  * `sysutils/ruby-puppet-syntax`: `7.2.0` -> `7.2.3`
  * `devel/ruby-json-schema` (PDK dependency): `6.2.0` -> `6.2.1`
* **Hiera-eyaml-gpg dependency**:
  * `security/ruby-gpgme`: `2.0.26` -> `2.0.27`
* **[r10k](https://github.com/puppetlabs/r10k) dependencies**:
  * `devel/ruby-colored2`: `4.0.3` -> `4.1.0`
  * `www/ruby-faraday`: `2.14.4` -> `2.15.1`
  * `www/ruby-faraday-net_http`: `3.4.4` -> `3.5.0`
* **[Puppetboard](https://github.com/voxpupuli/puppetboard) & Python base**:
  * `www/puppetboard`: `7.0.2` -> `8.1.0`
  * `devel/py-cachelib`: `0.17.0` -> `0.17.2`

{{< figure
    src="/images/puppetboard-in-action.png"
    alt="Puppetboard in action on OpenBSD"
    caption="Puppetboard in action on OpenBSD"
    class="ma0 w-75"
>}}

---

## Security Auditing & Digital Forensics

Another major block involved tools from the realms of web security, digital forensics, WiFi analysis, and Software Defined Radio (SDR).

### WPScan and Its Ruby Dependencies

**[WPScan](https://wpscan.com/)** is a specialized black-box security scanner for WordPress installations that detects known vulnerabilities in core versions, plugins, and themes. Updating to version `4.2.1` required bringing numerous underlying Ruby gems up to date:

* `security/wpscan`: `4.1.0` -> `4.2.1`
* `devel/ruby-activesupport`: `8.1.3.1` -> `8.1.4.1`
* `net/ruby-connection_pool`: `3.0.2` -> `3.0.3`
* `www/ruby-ethon`: `0.18.0` -> `0.19.2`
* `www/ruby-xmlrpc`: `0.3.4` -> `0.3.5`
* `www/ruby-ferrum`: `0.18.0` -> `0.21.0`
* `www/ruby-addressable`: `2.9.0` -> `2.10.0`

### Plaso (log2timeline) & Forensics Libraries

**[Plaso](https://github.com/log2timeline/plaso)** (also known as the backend behind *log2timeline*) is a powerful framework for digital forensics that extracts timestamps from a wide variety of filesystems, logs, and artifacts, combining them into a unified super-timeline. Naturally, such complex parser suites bring along their own libraries for filesystem abstraction and encryption:

* `security/plaso`: `20260720` -> `20261027`
* `security/py-dfdatetime`: `20260730` -> `20261027`
* `security/py-dfvfs`: `20260730` -> `20261027`
* `sysutils/libyal/libcaes`: `20251121` -> `20261028`
* `sysutils/libyal/libfsntfs`: `20260727` -> `20260827`
* `sysutils/afflib3`: `3.7.22` -> `3.7.23`
* `devel/py-parsing`: `3.2.3` -> `3.3.3`

### Kismet, hcxtools, and Other Security Tools

* **[Kismet](https://kismetwireless.net/)** (`net/kismet` `2025-09-R1` -> `2026-10-R1`): A widely used wireless network detector, sniffer, and intrusion detection system for WiFi, Bluetooth, SDR, and much more. You can find more about Kismet—including my original porting effort from the ancient Kismet version to the modern codebase—in my other blog posts [here](/en/tags/kismet/). Alongside the regular changes from the [Release Notes](https://kismetwireless.net/posts/kismet-2026-10-r1/), I contributed a new pull request upstream with build fixes as well as adjustments around SUID handling ([https://github.com/kismetwireless/kismet/pull/767](https://github.com/kismetwireless/kismet/pull/767))—and it is, of course, a special honor to be mentioned by name multiple times in the release notes!
  Furthermore, Stuart `sthen@` pointed out to me today that Kismet unwantedly picks up `-latomic` during compilation if `gcc-libs` is installed on the system. A quick look into `configure` and `configure.ac` quickly revealed the root cause: the script previously only checked *whether* linking against `-latomic` worked in principle, rather than whether it was actually *necessary*. Accordingly, I adjusted `configure.ac` to first test whether a program using atomic operations compiles and links without `-latomic`, only adding `-latomic` if it is genuinely needed. Upstream PR [here](https://github.com/kismetwireless/kismet/pull/775).

{{< figure
    src="/images/kismet_in_action.png"
    alt="Kismet in action on OpenBSD"
    caption="Kismet in action on OpenBSD"
    class="ma0 w-75"
>}}

Multiple data sources such as Wi-Fi, Zigbee, and ADS-B (see screenshot) now run noticeably smoother in parallel. In addition, Kismet has gained support for quite a few new data sources and hardware modules—including even gamma radiation detectors, list available [here](https://www.kismetwireless.net/docs/readme/datasources/datasources/). However, these new sources are still waiting for a hands-on test from my side. (**HINT HINT** anyone who wants to send hardware donations my way is welcome to reach out via comments or direct donations, see below ;)

* **[hcxtools](https://github.com/ZerBea/hcxtools)** (`security/hcxtools` `7.1.2` -> `8.0.0`): A collection of tools for converting and preparing WiFi capture files for penetration testing.

{{< admonition type=warning title="Breaking Change in hcxtools 8.0.0" >}}
With major release `8.0.0`, `hcxtools` introduces a modified hash file format. The generated hash files are **no longer compatible with older Hashcat versions**—so you should update your entire toolchain accordingly if needed.
{{< /admonition >}}

* **Other updates from the security & networking corner**:
  * `security/exploitdb`: `2026-09-12` -> `2026-11-23` (The local archive of known exploits and shellcodes)
  * `security/py-fickling`: `0.1.12` -> `0.1.13` (Static analyzer and decompiler for Python pickle files)
  * `net/proxychains-ng`: `4.17` -> `4.18` (Minor bugfix release for redirecting TCP connections via SOCKS/HTTP proxies)
  * `net/udns`: `0.6` -> `0.7` (Asynchronous DNS resolver library)

### SDR Tools (Software Defined Radio)

Fitting the topic of RF analysis, there was also a small update bundle around **Software Defined Radio (SDR)**—that is, receiving and decoding a wide variety of radio signals using inexpensive USB DVB-T sticks based on the RTL2832U chipset:

* `comms/rtl-sdr`: `2.0.2` -> `2.0.3` (The foundational driver and tool suite for operating RTL2832U sticks as general-purpose SDR receivers on OpenBSD)
* `comms/readsb`: `3.16.16` -> `3.16.21` (A high-performance ADS-B decoder for receiving and tracking aircraft transponder signals in real time)

---

## GNUstep, Smart Home & Miscellaneous

Away from server automation and security, there were also a few nice updates for desktop and home use:

* **SimpleAgenda** (`x11/gnustep/simpleagenda` `0.48` -> `0.50`): A lightweight calendar and task manager for the GNUstep desktop environment. Some adjustments were required here for compatibility between `libical` 3 and 4—I shared the corresponding patches right back upstream.
* **openHAB** (`misc/openhab` and `misc/openhab-addons` `5.2.1` -> `5.2.4`): The well-known vendor-independent smart home platform. The patch for this update came directly from the maintainer via the `ports@` mailing list.
* **Ruby HTTPClient** (`www/ruby-httpclient` `2.9.0` -> `2.11.0`): Another updated HTTP client library for Ruby.

---

## That's a Wrap Again

This update sprint once again demonstrated how tightly intertwined modern software ecosystems are: a single security fix or an update to a core Ruby gem quickly triggers a whole wave of follow-up updates. It is especially gratifying when not only packages on OpenBSD get updated in the process, but the upstream projects themselves benefit directly from pull requests—as was the case with Kismet or SimpleAgenda.

As we all know, the ports tree never stands still: the next Ruby gems, Python packages, and security advisories are surely already waiting around the corner. Until the next sprint!

