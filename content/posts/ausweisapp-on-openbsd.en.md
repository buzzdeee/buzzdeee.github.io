---
title: "AusweisApp on OpenBSD: When LibreSSL Says No, Wine Steps In"
subtitle: "A field report navigating failed ports, MSI extractions, and IPv4-to-IPv6 UDP broadcast bridges"
date: 2026-11-16T21:05:54+01:00
lastmod: 2026-11-16T21:05:54+01:00
draft: false
description: "How to get the German AusweisApp running on OpenBSD using 64-bit Wine and socat despite LibreSSL lacking RSA-PSK support."
license: "MIT"
images:
  - "/images/Ausweisapp_on_OpenBSD.jpg"
  - "/images/AusweisApp_Installer.png"
  - "/images/AusweisApp_SmartphonePairing.png"
  - "/images/AusweisApp_SmartphoneSettings.png"

tags:
  - OpenBSD
  - Wine
  - AusweisApp
  - eID
  - Networking
  - socat
categories:
  - OpenBSD
  - Tutorials

featuredImage: "/images/Ausweisapp_on_OpenBSD.jpg"
featuredImagePreview: "/images/Ausweisapp_on_OpenBSD.jpg"

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
    - "/images/Ausweisapp_on_OpenBSD.jpg"
    - "/images/AusweisApp_Installer.png"
    - "/images/AusweisApp_SmartphonePairing.png"
    - "/images/AusweisApp_SmartphoneSettings.png"
  # ...
---

## Introduction and Motivation

For years, it just sat there: that unassuming letter letting me know I could activate the online ID function on my German national identity card (*Personalausweis*). The idea behind it is genuinely appealing—no more booking an appointment at the local citizens' office for every minor errand, instead handling government services, ELSTER (tax filings), or BundID comfortably from my home desk.

When I finally found the time and leisure to look into it, however, my initial enthusiasm quickly gave way to disillusionment. As an OpenBSD user, you are used to occasionally rolling up your sleeves when software is primarily developed for Linux, Windows, or macOS. Yet the **[AusweisApp](https://de.wikipedia.org/wiki/AusweisApp_(Governikus))** proved to be a particularly stubborn candidate.

## The First Attempt: A Native OpenBSD Port

At first, the starting position didn't look too bad: I was delighted to discover that the source code for AusweisApp is freely available and that pre-built packages already exist for various Linux distributions. This made me cautiously optimistic that the application could be built for OpenBSD without too many gymnastics.

Back in December 2024, I tried porting the then-current version **2.4.0** of AusweisApp natively to OpenBSD. However, the endeavor failed spectacularly right during the build process due to cryptographic requirements:

{{< highlight text >}}
-- Could NOT find Qt6QmlCompilerPlusPrivateTools (missing: Qt6QmlCompilerPlusPrivateTools_DIR)
CMake Error at cmake/Libraries.cmake:183 (message):
  Your OpenSSL library looks incompatible:

  Change Dir:
  '/home/ports/pobj/amd64/AusweisApp-2.4.0/build-amd64/CMakeFiles/CMakeTmp'

  Run Build Command(s): /usr/local/bin/ninja -v cmTC_125bc

  [1/2] /home/ports/pobj/amd64/AusweisApp-2.4.0/bin/c++ -DQT_CORE_LIB
  -DQT_NETWORK_LIB -DQT_NO_DEBUG -isystem
  /usr/local/include/X11/qt6/QtNetwork -isystem /usr/local/include/X11/qt6
  -isystem /usr/local/include/X11/qt6/QtCore -isystem
  /usr/local/lib/qt6/mkspecs/openbsd-clang -O2 -pipe -g -O0 -fPIC -MD -MT
  CMakeFiles/cmTC_125bc.dir/openssl.cpp.o -MF
  CMakeFiles/cmTC_125bc.dir/openssl.cpp.o.d -o
  CMakeFiles/cmTC_125bc.dir/openssl.cpp.o -c
  /home/ports/pobj/amd64/AusweisApp-2.4.0/AusweisApp-2.4.0/cmake/tests/openssl.cpp

  FAILED: CMakeFiles/cmTC_125bc.dir/openssl.cpp.o
  ...
  /home/ports/pobj/amd64/AusweisApp-2.4.0/AusweisApp-2.4.0/cmake/tests/openssl.cpp:26:3:
  error: RSA-PSK is required.

     26 |         #error RSA-PSK is required.
        |          ^

  1 error generated.
  ninja: build stopped: subcommand failed.
{{< /highlight >}}

{{< admonition type="info" title="Why Building Against OpenSSL Doesn't Help" >}}
As is well known, OpenBSD relies on **LibreSSL**, which—for good reasons—does not support `RSA-PSK`. You might get the idea to simply compile AusweisApp against the separately available OpenSSL package from ports. However, since AusweisApp links against **Qt6**, and Qt6 on OpenBSD is in turn built against LibreSSL, mixing both SSL libraries leads to nasty crashes and symbol conflicts at runtime at the latest.
{{< /admonition >}}

At this point, I put the project aside in frustration.

## A Fresh Start: 64-Bit Wine on OpenBSD

Recently, however, the situation changed: **Wine** was imported into the OpenBSD ports tree—complete with support for **64-bit Windows applications**! That practically begged for a second attempt, this time using the [current Windows version of AusweisApp](https://www.ausweisapp.bund.de/download) (version 2.6.2).

### The MSI Installer Hurdle

Launching the `.msi` installer directly, Wine initially greeted me with a few harmless console messages, though it did manage to open a graphical window:

{{< highlight text >}}
00a4:err:ntoskrnl:ZwLoadDriver failed to create driver L"\\Registry\\Machine\\System\\CurrentControlSet\\Services\\winebth": c00000e5
003c:fixme:service:scmdatabase_autostart_services Auto-start service L"winebth" failed to start: 1359
00e4:fixme:file:NtLockFile I/O completion on lock not implemented yet
00e4:fixme:ntdll:NtQuerySystemInformation info_class SYSTEM_PERFORMANCE_INFORMATION
{{< /highlight >}}

The joy was short-lived: the installer stubbornly insisted that I needed at least 64-bit Windows 10 or Windows Server 2016.

{{< figure src="/images/AusweisApp_Installer.png" alt="Error message from the AusweisApp installer under Wine" caption="The MSI installer refuses to proceed due to an allegedly outdated Windows version." class="ma0 w-75" >}}

So I explicitly set the version to Windows 10 using `winecfg` and verified it via the registry:

{{< highlight shell >}}
winecfg -v win10
wine reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion" /v ProductName
{{< /highlight >}}

The output dutifully confirmed: `ProductName REG_SZ Microsoft Windows 10`. Nevertheless, the installer threw the exact same error message on the next launch.

Instead of continuing to argue with the installation routine over who knows better, an MSI package can fortunately just be extracted administratively and the application launched directly:

{{< highlight shell >}}
msiexec /a AusweisApp-2.6.2.msi /qb TARGETDIR="C:\AusweisApp"
wine ${HOME}/.wine/drive_c/AusweisApp/PFiles64/AusweisApp/AusweisApp.exe
{{< /highlight >}}

And lo and behold: AusweisApp started up without a single complaint!

### The Networking Puzzle During Smartphone Pairing

Since I don't use a dedicated USB smartcard reader, I wanted to pair my iPhone (with AusweisApp installed) as a card reader over the local network. Nothing easier than that! Or so I thought—in pairing mode, the desktop and smartphone simply refused to find each other.

A closer look using network diagnostics like `tcpdump`, `fstat -p`, as well as `netstat -f inet` and `netstat -f inet6` finally shed light on the mystery—and uncovered some curious behavior:

- The **iPhone** sends its discovery requests as an **IPv4 broadcast** on **UDP port 24727**.
- Meanwhile, **AusweisApp running under Wine** was listening exclusively on **IPv6 UDP port 24727**—and not on IPv4 at all!

Even explicitly disabling IPv6 on the network interface (`ifconfig rge0 -inet6`) couldn't convince the application to open an IPv4 socket.

{{< figure src="/images/AusweisApp_SmartphonePairing.png" alt="No smartphone to be found far and wide" caption="No smartphone to be found far and wide" class="ma0 w-75" >}}

### The Solution: Bridging IPv4 and IPv6

If the app refuses to listen on IPv4, the packets simply have to be routed to where it *is* listening. First, I wrote a small Python script that accepts incoming IPv4 broadcasts on port 24727 and forwards them to `[::1]:24727`.

However, the problem can be solved much more elegantly as a one-liner using **socat**, without needing any extra script:

{{< highlight shell >}}
socat UDP4-RECVFROM:24727,broadcast,fork UDP6-SENDTO:[::1]:24727
{{< /highlight >}}

{{< admonition type="tip" title="A Handy Alias for Everyday Use" >}}
To avoid having to open two terminals every time, adding a small alias to your shell configuration (e.g., `~/.profile`) comes in very handy:

`alias ausweisapp="socat UDP4-RECVFROM:24727,broadcast,fork UDP6-SENDTO:[::1]:24727 & wine ${HOME}/.wine/drive_c/AusweisApp/PFiles64/AusweisApp/AusweisApp.exe"`

*(Note: For communication between the smartphone and desktop to succeed, your firewall—i.e., `pf`—must allow incoming traffic on **UDP port 24727**.)*
{{< /admonition >}}

Once the forwarding was running, pairing with the iPhone worked right out of the box! I was able to change my transport PIN, read out my ID card data, and subsequently log in to **BundID**-secured services as well as **ELSTER** directly via Firefox on OpenBSD.

{{< figure src="/images/AusweisApp_SmartphoneSettings.png" alt="Pairing finally succeeded." caption="Pairing finally succeeded." class="ma0 w-75" >}}

## Summary and Outlook

It certainly takes a small detour via Wine and a somewhat clunky workaround with `socat`, but at the end of the day, the result is what counts: the online ID function now works reliably on OpenBSD. One minor caveat remains, though: the Wine process for AusweisApp permanently hogs roughly 100% of a single CPU core even while idling. Since you only need the application briefly for the authentication process itself and can close it right afterward, that's easy enough to live with in day-to-day use.

I am currently checking on the `ports@` mailing list to see whether this networking quirk can be fixed directly in Wine on OpenBSD, or if it simply comes down to how AusweisApp itself opens its socket—which would obviously limit our options.

And there might even be hope for a much cleaner solution down the road: `kirill@` reached out to me mentioning he might still have an older, half-finished port of AusweisApp lying around where he had already addressed that exact `RSA-PSK` issue. Let's see if that eventually leads to a genuine native port for OpenBSD. Time will tell!

