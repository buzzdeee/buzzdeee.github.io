---
title: "Soekris net5501 and OpenBSD"
subtitle: "The Legendary Embedded Classic in a Long-Term Test"
date: 2026-09-16T21:30:00+02:00
lastmod: 2026-09-16T21:30:00+02:00
draft: false
description: "A nostalgic and factual retrospective on the Soekris net5501 as an extremely power-efficient OpenBSD platform for network and console services."
license: "MIT"
images: [ "/images/Soekris-Front.jpg", "/images/Soekris-Back.jpg", "/images/Soekris-Innen.jpg", "Sunix-8-Port-Serial.jpg" ]

tags: ["OpenBSD", "hardware", "soekris", "embedded", "server", "homelab", "i386"]
categories: ["Hardware", "OpenBSD"]

featuredImage: "/images/Soekris-Front.jpg"
featuredImagePreview: "/images/Soekris-Front.jpg"

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
seo:
  images: [ "/images/Soekris-Front.jpg", "/images/Soekris-Back.jpg", "/images/Soekris-Innen.jpg", "Sunix-8-Port-Serial.jpg" ]
---

Anyone who has been around the world of Unix security firewalls and minimalist server systems for a while remembers the grey-green or dark cases from Soekris Engineering with a certain nostalgia. For years, the **[Soekris net5501](https://ebay.us/IQQupe#affiliate)** was the gold standard for highly reliable, fanless embedded routers and network appliances running OpenBSD.

After writing a while ago about the [Fujitsu Futro S920 and OpenBSD](/fujitso-futro-s920-openbsd/), today we take a look at a real veteran of the x86 32-bit era. Although Soekris Engineering unfortunately closed its doors years ago, the hardware just keeps running and running—even if it has moved into enthusiast status today.

<figure class="ma0 w-75">
  <img src="/images/Soekris-Back.jpg" alt="Soekris net5501 Rear View">
  <figcaption>The Soekris net5501: Rear View</figcaption>
</figure>

---

## Hardware Features and Interfaces

The net5501 relies on the AMD Geode LX SOC—a platform that operates extremely energy-efficiently, though in terms of raw performance, it is naturally worlds apart from modern hardware. In return, the board shines with its expandability and fantastic reliability during continuous operation.

{{< admonition type="info" title="Hardware Overview & Specifications" >}}
* **Processor:** AMD Geode LX 800 (500 MHz x86, i386 architecture)
* **RAM:** 512 MB DDR-SDRAM (soldered onboard)
* **Networking:** 4x Fast Ethernet ports (`vr0` through `vr3`, VIA VT6105M Rhine III)
* **Storage:** CompactFlash slot (internal, serves as boot medium/hard drive) as well as a primary IDE channel
* **Serial Ports:** Standard console port via RS-232 (`com0`), expandable via internal headers or PCI cards
* **Expandability:** 1x 32-bit PCI slot and MiniPCI slot (e.g., for crypto cards, Wi-Fi, or multiplex serial cards)
* **Crypto Acceleration:** Onboard AMD Geode LX Crypto Engine (`glxsb0`) for hardware RNG and AES encryption
* **Power Consumption:** A phenomenal **4 to 6 Watts** power draw via a simple external power supply (or built-in, as seen here in the rackmount variant)
{{< /admonition >}}

<figure class="ma0 w-75">
  <img src="/images/Soekris-Innen.jpg" alt="Soekris net5501 Interior View">
  <figcaption>Inside the Soekris net5501, including the CompactFlash card</figcaption>
</figure>



---

## Hardware Recognition under OpenBSD (i386)

Even though the 32-bit i386 architecture is gradually being abandoned by many Linux distributions nowadays, the OpenBSD project remains true to the legacy silicon. The net5501 runs flawlessly using the standard i386 kernel.

Here is an excerpt from the boot log (`dmesg`) under OpenBSD 8.0-beta:

{{< highlight text >}}
[ using 2246276 bytes of bsd ELF symbol table ]
Copyright (c) 1982, 1986, 1989, 1991, 1993
        The Regents of the University of California.  All rights reserved.
Copyright (c) 1995-2026 OpenBSD. All rights reserved.  https://www.OpenBSD.org

OpenBSD 8.0-beta (GENERIC) #121: Tue Sep 15 12:58:53 MDT 2026
    deraadt@i386.openbsd.org:/usr/src/sys/arch/i386/compile/GENERIC
real mem  = 536363008 (511MB)
avail mem = 508542976 (484MB)
random: good seed from bootblocks
mpath0 at root
scsibus0 at mpath0: 256 targets
mainbus0 at root
bios0 at mainbus0: date 20/80/26, BIOS32 rev. 0 @ 0xfac40
pcibios0 at bios0: rev 2.0 @ 0xf0000/0x10000
pcibios0: pcibios_get_intr_routing - function not supported
pcibios0: PCI IRQ Routing information unavailable.
pcibios0: PCI bus #0 is the last bus
bios0: ROM list: 0xc8000/0xa800
cpu0 at mainbus0: (uniprocessor)
cpu0: Geode(TM) Integrated Processor by AMD PCS ("AuthenticAMD" 586-class) 500 MHz, 05-0a-02
cpu0: FPU,DE,PSE,TSC,MSR,CX8,SEP,PGE,CMOV,CFLUSH,MMX,MMXX,3DNOW2,3DNOW
mtrr: K6-family MTRR support (2 registers)
amdmsr0 at mainbus0
pci0 at mainbus0 bus 0: configuration mode 1 (bios)
0:20:0: io address conflict 0x6100/0x100
0:20:0: io address conflict 0x6200/0x200
pchb0 at pci0 dev 1 function 0 "AMD Geode LX" rev 0x31
glxsb0 at pci0 dev 1 function 2 "AMD Geode LX Crypto" rev 0x00: RNG AES
vr0 at pci0 dev 6 function 0 "VIA VT6105M RhineIII" rev 0x96: irq 11, address 00:00:24:c9:d4:98
ukphy0 at vr0 phy 1: Generic IEEE 802.3u media interface, rev. 3: OUI 0x004063, model 0x0034
vr1 at pci0 dev 7 function 0 "VIA VT6105M RhineIII" rev 0x96: irq 5, address 00:00:24:c9:d4:99
ukphy1 at vr1 phy 1: Generic IEEE 802.3u media interface, rev. 3: OUI 0x004063, model 0x0034
vr2 at pci0 dev 8 function 0 "VIA VT6105M RhineIII" rev 0x96: irq 9, address 00:00:24:c9:d4:9a
ukphy2 at vr2 phy 1: Generic IEEE 802.3u media interface, rev. 3: OUI 0x004063, model 0x0034
vr3 at pci0 dev 9 function 0 "VIA VT6105M RhineIII" rev 0x96: irq 12, address 00:00:24:c9:d4:9b
ukphy3 at vr3 phy 1: Generic IEEE 802.3u media interface, rev. 3: OUI 0x004063, model 0x0034
puc0 at pci0 dev 14 function 0 "Sunix 40XX" rev 0x01: ports: 16 com
com4 at puc0 port 0 irq 10: ti16750, 64 byte fifo
com4: probed fifo depth: 32 bytes
com5 at puc0 port 1 irq 10: ti16750, 64 byte fifo
com5: probed fifo depth: 32 bytes
com6 at puc0 port 2 irq 10: ti16750, 64 byte fifo
com6: probed fifo depth: 32 bytes
com7 at puc0 port 3 irq 10: ti16750, 64 byte fifo
com7: probed fifo depth: 32 bytes
com8 at puc0 port 4 irq 10: ti16750, 64 byte fifo
com8: probed fifo depth: 32 bytes
com9 at puc0 port 5 irq 10: ti16750, 64 byte fifo
com9: probed fifo depth: 32 bytes
com10 at puc0 port 6 irq 10: ti16750, 64 byte fifo
com10: probed fifo depth: 32 bytes
com11 at puc0 port 7 irq 10: ti16750, 64 byte fifo
com11: probed fifo depth: 32 bytes
glxpcib0 at pci0 dev 20 function 0 "AMD CS5536 ISA" rev 0x03: rev 3, 32-bit 3579545Hz timer, watchdog, gpio, i2c
gpio0 at glxpcib0: 32 pins
iic0 at glxpcib0
pciide0 at pci0 dev 20 function 2 "AMD CS5536 IDE" rev 0x01: DMA, channel 0 wired to compatibility, channel 1 wired to compatibility
wd0 at pciide0 channel 0 drive 1: <SanDisk SDCFH-002G>
wd0: 1-sector PIO, LBA, 1918MB, 3928176 sectors
wd0(pciide0:0:1): using PIO mode 4, DMA mode 2
pciide0: channel 1 ignored (disabled)
ohci0 at pci0 dev 21 function 0 "AMD CS5536 USB" rev 0x02: irq 15, version 1.0, legacy support
ehci0 at pci0 dev 21 function 1 "AMD CS5536 USB" rev 0x02: irq 15
usb0 at ehci0: USB revision 2.0
uhub0 at usb0 configuration 1 interface 0 "AMD EHCI root hub" rev 2.00/1.00 addr 1
isa0 at glxpcib0
isadma0 at isa0
com0 at isa0 port 0x3f8/8 irq 4: ns16550a, 16 byte fifo
com0: console
com1 at isa0 port 0x2f8/8 irq 3: ns16550a, 16 byte fifo
pckbc0 at isa0 port 0x60/5 irq 1 irq 12
pckbc0: unable to establish interrupt for irq 12
pckbd0 at pckbc0 (kbd slot)
wskbd0 at pckbd0: console keyboard
pcppi0 at isa0 port 0x61
spkr0 at pcppi0
nsclpcsio0 at isa0 port 0x2e/2: NSC PC87366 rev 10: GPIO VLM TMS
gpio1 at nsclpcsio0: 29 pins
npx0 at isa0 port 0xf0/16: reported by CPUID; using exception 16
usb1 at ohci0: USB revision 1.0
uhub1 at usb1 configuration 1 interface 0 "AMD OHCI root hub" rev 1.00/1.00 addr 1
vscsi0 at root
scsibus1 at vscsi0: 256 targets
softraid0 at root
scsibus2 at softraid0: 256 targets
root on wd0a (2e84bed4ba506bce.a) swap on wd0b dump on wd0b
{{< /highlight >}}

{{< admonition type="note" title="Serial Console as Default" >}}
Soekris hardware does not feature a graphical video output (VGA/HDMI). Management is handled purely via serial on `com0` (9600 baud by default in Soekris comBIOS—though it can be bumped up to 115200 baud). In OpenBSD, this needs to be configured accordingly in `/etc/boot.conf` and `/etc/ttys`.
{{< /admonition >}}

{{< admonition type="warning" title="Important Note: Reducing Write Cycles on CompactFlash" >}}
Since the system shown runs on a 2 GB CompactFlash card (`wd0`: SanDisk SDCFH-002G), you should be EXTREMELY cautious with frequent write operations to prevent premature wear on the flash media.

**Solution:** Mount directories like `/var/log`, `/tmp`, or `/var/run` in RAM via MFS (Memory File System) or forward log entries directly to a remote log server via `syslogd`. Alternatively, you can also install a native SATA hard drive or SSD.
{{< /admonition >}}

---

## Retrospective Use Cases

Given the modest 500 MHz clock rate and 100 Mbit/s network interfaces (`vr0`–`vr3`), the Soekris was primarily responsible for specialized administrative tasks in my network:

1. **Serial Terminal Server:** Thanks to the installed Sunix PCI card, a full 8 additional RS-232 serial interfaces (`com4` through `com11`) were available. Perfect for centrally accessing the console ports of switches, routers, or other headless servers via `cu`.
2. **Out-of-Band Management:** Operating as a small, resilient emergency access node.
3. **Network Server:** Serving core infrastructure protocols like DNS, DHCP, TFTP, etc.

A special highlight of my setup was using a [Sunix Multi-Port Serial PCI Expansion Card](https://ebay.us/I3tMXl#affiliate) (`puc0`), turning the unit into the ultimate multi-port console server.

<figure class="ma0 w-75">
  <img src="/images/Sunix-8-Port-Serial.jpg" alt="Sunix 8 Port Serial PCI Card">
  <figcaption>Sunix 8 Port Serial PCI Card</figcaption>
</figure>

---

## Conclusion & Outlook

Is a [Soekris net5501](https://ebay.us/IQQupe#affiliate) still practical in this day and age? For modern gigabit connections with deep packet inspection (PF), the Geode LX 800 is simply too slow. The lack of 64-bit support and gigabit interfaces also clearly shows the platform's age.

However, if you are looking for a stoic, completely silent piece of hardware history for minimalist tasks or as a dedicated serial server, you can still occasionally find used units on marketplaces like [eBay](https://ebay.us/EqGEBs#affiliate). It is always impressive to see how little computing power and memory a full-featured, highly secure OpenBSD system actually needs to stay useful!

