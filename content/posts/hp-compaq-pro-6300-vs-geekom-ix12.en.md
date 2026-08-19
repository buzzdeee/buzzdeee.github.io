---
title: "OpenBSD Upgrades: My Switch from HP Compaq Pro to the Silent Geekom IX12"
subtitle: "Less power, zero noise, and real-world performance insights."
date: 2026-08-17T16:50:10+02:00
lastmod: 2026-08-17T16:50:10+02:00
draft: false
author: ""
authorLink: ""
description: "A field report on migrating from an aging HP Compaq 6300 desktop to a fanless Geekom IX12 Mini-PC running OpenBSD, including benchmark comparisons during compilation."
license: "MIT"
images: [ "/images/hp_vs_geekom.jpg", "/images/geekom_unboxed.jpeg", "/images/geekom_front.jpeg", "/images/geekom_back.jpeg", "/images/geekom_open.jpeg", "/images/geekom_open_no_disk.jpeg" ]

tags: ["OpenBSD", "Geekom", "Hardware", "Benchmark", "MiniPC"]
categories: ["Tech", "Hardware"]

featuredImage: "/images/hp_vs_geekom.jpg"
featuredImagePreview: "/images/hp_vs_geekom.jpg"

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
share:
  enable: true
comment:
  enable: true
seo:
  images: [ "/images/hp_vs_geekom.jpg", "/images/geekom_unboxed.jpeg", "/images/geekom_front.jpeg", "/images/geekom_back.jpeg", "/images/geekom_open.jpeg", "/images/geekom_open_no_disk.jpeg" ]
---

A reliable workhorse is a fine thing. For years, an aging desktop drudge performed faithful service for daily internet browsing, small administrative tasks, and the occasional compilation of ports under OpenBSD. It is an **HP Compaq Pro 6300 SFF** (manufactured around **2012**).

{{< admonition type="note" title="The old companion: HP Compaq Pro 6300 SFF" open=false >}}
The HP Compaq Pro 6300 SFF was a typical business workhorse PC around 2012. Equipped with an Intel Core i5-3470 (Ivy Bridge, 4 cores, 3.20 GHz) and 32 GB of DDR3 RAM, it continued to perform its duties without complaint even after more than a decade, though naturally, it excelled neither in energy efficiency nor in noise generation.
{{< /admonition >}}

Here is the complete kernel log (`dmesg`) of the old HP system for reference:

{{< highlight text >}}
OpenBSD 7.9-current (GENERIC.MP) #4: Sun Jun 21 12:53:34 MDT 2026
    deraadt@amd64.openbsd.org:/usr/src/sys/arch/amd64/compile/GENERIC.MP
real mem = 34279374848 (32691MB)
avail mem = 33212223488 (31673MB)
random: good seed from bootblocks
mpath0 at root
scsibus0 at mpath0: 256 targets
mainbus0 at root
bios0 at mainbus0: SMBIOS rev. 2.7 @ 0xe928b (60 entries)
bios0: vendor Hewlett-Packard version "K01 v02.05" date 05/07/2012
bios0: Hewlett-Packard HP Compaq Pro 6300 SFF
acpi0 at bios0: ACPI 4.0
acpi0: sleep states S0 S3 S4 S5
acpi0: tables DSDT FACP APIC MCFG HPET SSDT SSDT SSDT SSDT TCPA ASF!
acpi0: wakeup devices PS2K(S3) PS2M(S3) P0P1(S4) USB1(S3) USB2(S3) USB3(S3) USB4(S3) USB5(S3) USB6(S3) USB7(S3) RP01(S4) PXSX(S4) RP02(S4) PXSX(S4) RP03(S4) PXSX(S4) [...]
acpitimer0 at acpi0: 3579545 Hz, 24 bits
acpimadt0 at acpi0 addr 0xfee00000: PC-AT compat
cpu0 at mainbus0: apid 0 (boot processor)
cpu0: Intel(R) Core(TM) i5-3470 CPU @ 3.20GHz, 3192.95 MHz, 06-3a-09, patch 00000021
cpu0: cpuid 1 edx=bfebfbff<FPU,VME,DE,PSE,TSC,MSR,PAE,MCE,CX8,APIC,SEP,MTRR,PGE,MCA,CMOV,PAT,PSE36,CFLUSH,DS,ACPI,MMX,FXSR,SSE,SSE2,SS,HTT,TM,PBE> ecx=77bae3ff<SSE3,PCLMUL,DTES64,MWAIT,DS-CPL,VMX,SMX,EST,TM2,SSSE3,CX16,xTPR,PDCM,PCID,SSE4.1,SSE4.2,x2APIC,POPCNT,DEADLINE,AES,XSAVE,AVX,F16C,RDRAND>
cpu0: cpuid 6 eax=77<SENSOR,ARAT,PTS> ecx=9<EFFFREQ>
cpu0: cpuid 7.0 ebx=281<FSGSBASE,SMEP,ERMS> edx=9c000400<MD_CLEAR,IBRS,IBPB,STIBP,L1DF,SSBD>
cpu0: cpuid a vers=3, gp=8, gpwidth=48, ff=3, ffwidth=48
cpu0: cpuid d.1 eax=1<XSAVEOPT>
cpu0: cpuid 80000001 edx=28100800<NXE,RDTSCP,LONG> ecx=1<LAHF>
cpu0: cpuid 80000007 edx=100<ITSC>
cpu0: MELTDOWN
cpu0: 32KB 64b/line 8-way D-cache, 32KB 64b/line 8-way I-cache, 256KB 64b/line 8-way L2 cache, 6MB 64b/line 12-way L3 cache
cpu0: smt 0, core 0, package 0, type P
mtrr: Pentium Pro MTRR support, 10 var ranges, 88 fixed ranges
cpu0: apic clock running at 99MHz
cpu0: mwait min=64, max=64, C-substates=0.2.1.1, IBE
cpu1 at mainbus0: apid 2 (application processor)
cpu1: Intel(R) Core(TM) i5-3470 CPU @ 3.20GHz, 3192.91 MHz, 06-3a-09, patch 00000021
cpu1: smt 0, core 1, package 0, type P
cpu2 at mainbus0: apid 4 (application processor)
cpu2: Intel(R) Core(TM) i5-3470 CPU @ 3.20GHz, 3193.04 MHz, 06-3a-09, patch 00000021
cpu2: smt 0, core 2, package 0, type P
cpu3 at mainbus0: apid 6 (application processor)
cpu3: Intel(R) Core(TM) i5-3470 CPU @ 3.20GHz, 3193.16 MHz, 06-3a-09, patch 00000021
cpu3: smt 0, core 3, package 0, type P
ioapic0 at mainbus0: apid 2 pa 0xfec00000, version 20, 24 pins
acpimcfg0 at acpi0
acpimcfg0: addr 0xf8000000, bus 0-63
acpihpet0 at acpi0: 14318179 Hz
acpiprt0 at acpi0: bus 0 (PCI0)
acpiprt1 at acpi0: bus 2 (P0P1)
acpiprt2 at acpi0: bus -1 (RP01)
acpiprt3 at acpi0: bus -1 (RP02)
acpiprt4 at acpi0: bus -1 (RP03)
acpiprt5 at acpi0: bus -1 (RP04)
acpiprt6 at acpi0: bus -1 (RP05)
acpiprt7 at acpi0: bus -1 (RP06)
acpiprt8 at acpi0: bus -1 (RP07)
acpiprt9 at acpi0: bus -1 (RP08)
acpiprt10 at acpi0: bus 1 (PEG0)
acpiprt11 at acpi0: bus -1 (PEG1)
acpiprt12 at acpi0: bus -1 (PEG2)
acpiprt13 at acpi0: bus -1 (PEG3)
acpiec0 at acpi0: not present
acpipci0 at acpi0 PCI0: 0x00000010 0x00000011 0x00000000
acpicmos0 at acpi0
"PNP0303" at acpi0 not configured
"PNP0F03" at acpi0 not configured
com0 at acpi0 UAR1 addr 0x3f8/0x8 irq 4: ns16550a, 16 byte fifo
tpm0 at acpi0 TPM_ 1.2 (TIS) addr 0xfed40000/0x5000, Infineon SLB9635 1.2 rev 0x10
acpibtn0 at acpi0: PWRB(wakeup)
"PNP0C14" at acpi0 not configured
"PNP0C0B" at acpi0 not configured
"PNP0C0B" at acpi0 not configured
"PNP0C0B" at acpi0 not configured
"PNP0C0B" at acpi0 not configured
"PNP0C0B" at acpi0 not configured
acpicpu0 at acpi0: C3(350@80 mwait.1@0x20), C2(500@59 mwait.1@0x10), C1(1000@1 mwait.1), PSS
acpicpu1 at acpi0: C3(350@80 mwait.1@0x20), C2(500@59 mwait.1@0x10), C1(1000@1 mwait.1), PSS
acpicpu2 at acpi0: C3(350@80 mwait.1@0x20), C2(500@59 mwait.1@0x10), C1(1000@1 mwait.1), PSS
acpicpu3 at acpi0: C3(350@80 mwait.1@0x20), C2(500@59 mwait.1@0x10), C1(1000@1 mwait.1), PSS
acpipwrres0 at acpi0: FN00, resource for FAN0
acpipwrres1 at acpi0: FN01, resource for FAN1
acpipwrres2 at acpi0: FN02, resource for FAN2
acpipwrres3 at acpi0: FN03, resource for FAN3
acpipwrres4 at acpi0: FN04, resource for FAN4
acpitz0 at acpi0
acpitz0: critical temperature is 105 degC
acpitz1 at acpi0
acpitz1: critical temperature is 105 degC
acpivideo0 at acpi0: PEG0
acpivideo1 at acpi0: VGA_
acpivout0 at acpivideo1: LCD_
acpivideo2 at acpi0: GFX0
acpivout1 at acpivideo2: DD02
acpivout2 at acpivideo2: LCD_
cpu0: using VERW MDS workaround (except on vmm entry)
cpu0: Enhanced SpeedStep 3192 MHz: speeds: 3201, 3200, 3100, 3000, 2900, 2700, 2600, 2500, 2400, 2300, 2200, 2100, 1900, 1800, 1700, 1600 MHz
pci0 at mainbus0 bus 0
pchb0 at pci0 dev 0 function 0 "Intel Core 3G Host" rev 0x09
ppb0 at pci0 dev 1 function 0 "Intel Core 3G PCIE" rev 0x09: msi
pci1 at ppb0 bus 1
amdgpu0 at pci1 dev 0 function 0 "ATI Polaris 12" rev 0xc7
drm0 at amdgpu0
amdgpu0: msi
azalia0 at pci1 dev 0 function 1 "ATI Radeon Pro Audio" rev 0x00: msi
azalia0: no supported codecs
xhci0 at pci0 dev 20 function 0 "Intel 7 Series xHCI" rev 0x04: msi, xHCI 1.0
usb0 at xhci0: USB revision 3.0
uhub0 at usb0 configuration 1 interface 0 "Intel xHCI root hub" rev 3.00/1.00 addr 1
"Intel 7 Series MEI" rev 0x04 at pci0 dev 22 function 0 not configured
puc0 at pci0 dev 22 function 3 "Intel 7 Series KT" rev 0x04: ports: 16 com
com4 at puc0 port 0 apic 2 int 19: ns16550a, 16 byte fifo
com4: probed fifo depth: 0 bytes
em0 at pci0 dev 25 function 0 "Intel 82579LM" rev 0x04: msi, address 88:51:fb:69:14:84
ehci0 at pci0 dev 26 function 0 "Intel 7 Series USB" rev 0x04: apic 2 int 16
usb1 at ehci0: USB revision 2.0
uhub1 at usb1 configuration 1 interface 0 "Intel EHCI root hub" rev 2.00/1.00 addr 1
azalia1 at pci0 dev 27 function 0 "Intel 7 Series HD Audio" rev 0x04: msi
azalia1: codecs: Realtek ALC221
audio0 at azalia1
ehci1 at pci0 dev 29 function 0 "Intel 7 Series USB" rev 0x04: apic 2 int 23
usb2 at ehci1: USB revision 2.0
uhub2 at usb2 configuration 1 interface 0 "Intel EHCI root hub" rev 2.00/1.00 addr 1
ppb1 at pci0 dev 30 function 0 "Intel 82801BA Hub-to-PCI" rev 0xa4
pci2 at ppb1 bus 2
pcib0 at pci0 dev 31 function 0 vendor "Intel", unknown product 0x1e48 rev 0x04
ahci0 at pci0 dev 31 function 2 "Intel 7 Series AHCI" rev 0x04: msi, AHCI 1.3
ahci0: port 0: 6.0Gb/s
ahci0: port 1: 3.0Gb/s
ahci0: port 2: 1.5Gb/s
scsibus1 at ahci0: 32 targets
sd0 at scsibus1 targ 0 lun 0: <ATA, SSDPR-CX400-512-, SBFM> t10.ATA_SSDPR-CX400-512-G2_GZ6069394_
sd0: 488386MB, 512 bytes/sector, 1000215216 sectors, thin
sd1 at scsibus1 targ 1 lun 0: <ATA, WDC WD10EZRX-00A, 01.0> naa.50014ee206ab2f90
sd1: 953869MB, 512 bytes/sector, 1953525168 sectors
cd0 at scsibus1 targ 2 lun 0: <hp, CDDVDW SH-216BB, HE50> removable
ichiic0 at pci0 dev 31 function 3 "Intel 7 Series SMBus" rev 0x04: apic 2 int 18
iic0 at ichiic0
spdmem0 at iic0 addr 0x50: 8GB DDR3 SDRAM PC3-12800
spdmem1 at iic0 addr 0x51: 8GB DDR3 SDRAM PC3-12800
spdmem2 at iic0 addr 0x52: 8GB DDR3 SDRAM PC3-12800
spdmem3 at iic0 addr 0x53: 8GB DDR3 SDRAM PC3-12800
isa0 at pcib0
isadma0 at isa0
pckbc0 at isa0 port 0x60/5 irq 1 irq 12
pckbd0 at pckbc0 (kbd slot)
wskbd0 at pckbd0: console keyboard
pcppi0 at isa0 port 0x61
spkr0 at pcppi0
vmm0 at mainbus0: VMX/EPT
uhub3 at uhub1 port 1 configuration 1 interface 0 "Intel Rate Matching Hub" rev 2.00/0.00 addr 2
uhub4 at uhub2 port 1 configuration 1 interface 0 "Intel Rate Matching Hub" rev 2.00/0.00 addr 2
vscsi0 at root
scsibus2 at vscsi0: 256 targets
softraid0 at root
scsibus3 at softraid0: 256 targets
root on sd0a (70b482adfc8af4a4.a) swap on sd0b dump on sd0b
amdgpu0: POLARIS12 8 CU rev 0x00
amdgpu0: 3440x1440, 32bpp
wsdisplay0 at amdgpu0 mux 1: console (std, vt100 emulation), using wskbd0
wsdisplay0: screen 1-5 added (std, vt100 emulation)
{{< /highlight >}}

## The New Guy in the Office

A replacement had to be found! The objectives were clearly defined: **Fanless (passively cooled), energy-efficient, and sufficiently powerful for daily OpenBSD use.** After some searching, the choice fell on the **[Geekom IX12](https://www.awin1.com/cread.php?awinmid=32589&awinaffid=3025379&ued=https%3A%2F%2Fwww.geekom.de%2Fgeekom-ix12-luefterloser-mini-pc%2F#affiliate)**. Over the last few years, I have replaced various computers and switches; the last remaining device with a fan that is regularly switched on is my desktop. A "fanless" office is a fine thing; I wonder how I put up with it before, at times even with computers with broken fans that were quite annoying. But perhaps one just gets older ;)

---

## The Hardware Under the Microscope

### Unboxing the Geekom IX12

Right from the initial unboxing, the Geekom IX12 impresses with a high-quality, anodized housing. Its massive cooling fins dissipate the heat from the Intel N95 completely without fans. Factory-installed, the device comes with 8 GB of RAM and a 256 GB M.2 SATA SSD. Not incredibly lavish, but entirely sufficient for a firewall/router or network server. After all, it is also advertised by Geekom as "The ultimate firewall & gateway solution."

The scope of delivery is generous: a solid power supply unit with adapters, various cables, a VESA mount, and external antennas are included. The connections offer everything one could wish for: no less than **four 2.5G network ports**, a USB-C console port, five additional USB ports, as well as one HDMI and one DisplayPort output each.

{{< figure src="/images/geekom_unboxed.jpeg" alt="Geekom IX12 scope of delivery and connections" caption="The Geekom IX12 shines with an anodized housing, cooling fins, and plenty of accessories." class="ma0 w-75" >}}

{{< admonition type="note" title="Perfect for the Home Lab" open=true >}}
In particular, the combination of four full-fledged 2.5 Gigabit network connections, a console port, and passive cooling makes the device a fantastic platform for routers, OpenBSD firewalls, or edge nodes. However, Windows 11 is pre-installed at the factory – which was, of course, quickly replaced by OpenBSD! ;)
{{< /admonition >}}

### External Hardware

Externally, it makes a very solid impression, with a sturdy housing and ports at the front and back.

{{< figure src="/images/geekom_front.jpeg" alt="Geekom IX12 Mini PC Front" caption="The Geekom IX12 front view – USB, Display, SIM card, audio, and power button." class="ma0 w-75" >}}
{{< figure src="/images/geekom_back.jpeg" alt="Geekom IX12 Mini PC Rear" caption="The Geekom IX12 rear view – 4x LAN, console, power connection." class="ma0 w-75" >}}

### Internal Hardware

Turn the device upside down, remove the 4 small screws, and the cover can already be removed, giving very easy access to all replaceable parts. RAM, SATA M.2 SSD, WiFi, etc. – nothing is soldered, but everything is easily accessible and replaceable, just as one would wish.

{{< figure src="/images/geekom_open.jpeg" alt="Geekom IX12 Mini opened" caption="The Geekom IX12 opened." class="ma0 w-75" >}}
{{< figure src="/images/geekom_open_no_disk.jpeg" alt="Geekom IX12 opened, RAM and disk removed" caption="Geekom IX12 opened, RAM and disk removed." class="ma0 w-75" >}}

### Hardware Upgrades

For a full-fledged system as a desktop and for compiling ports, it needed to be a bit more. Therefore, the mini-PC received an upgrade:

* **Memory (RAM):** Upgraded from 8 GB to **[32 GB RAM](https://ebay.us/fmXn5I#affiliate)**
* **Storage:** Replacement of the 256 GB SSD with a **512 GB model**

To check the influence of the storage medium, two M.2 SATA SSDs were used for testing: a **[Verbatim Vi560](https://ebay.us/JKLBbx#affiliate)** and a **[Samsung EVO SSD 860](https://ebay.us/AjyRqF#affiliate)**. Both use the SATA interface but differ in controller and cache behavior under sustained I/O load.

{{< admonition type="note" title="Comparing Two M.2 SATA Disks" open=true >}}
Important to mention: **Both test candidates are M.2 SATA SSDs.** Although they use the same form factor and the same bus interface, exciting differences emerge in detail:
* **[Verbatim Vi560 (M.2 SATA)](https://ebay.us/JKLBbx#affiliate):** An affordable entry-level all-rounder. Solid for short accesses, its leaner cache management reaches performance limits more quickly during sustained high write and read cycles (such as when compiling).
* **[Samsung 860 EVO SSD (M.2 SATA)](https://ebay.us/AjyRqF#affiliate):** A proven classic in M.2 guise. Thanks to a mature controller and a better DRAM cache, it keeps transfer rates extremely stable even under continuous I/O load.
{{< /admonition >}}

Memory and SSD are cheap on eBay, see related links above.

{{< admonition type="warning" title="M.2 SATA vs. M.2 NVMe PCIe: Make Sure You Buy the Right Drive for Your Slot!" open=true >}}
Before purchasing a new M.2 SSD, always double-check the exact standards supported by your motherboard or laptop's M.2 slot. Even though the modules may look similar, they use vastly different protocols and interfaces. **Note on the tested device:** The Geekom IX12 tested here features an M.2 SATA 2280 slot and does not support NVMe PCIe SSDs!

* **M.2 SATA (AHCI):**
  * **Speed:** Maxes out at ~550 MB/s (limited by the SATA-III bus limit).
  * **Keying:** Usually features a **B+M-Key** (two small notches).
  * **Best for:** Older systems, specific mini PCs (such as the Geekom IX12), or budget storage upgrades.

* **M.2 NVMe (PCIe):**
  * **Speed:** Extremely fast (ranging from 3,500 to over 7,000 MB/s via PCIe 3.0/4.0/5.0).
  * **Keying:** Usually features an **M-Key** (a single notch).
  * **Best for:** Modern PCs, laptops, high-performance gaming, and workstations.

---

### What to Look Out For Before Buying:

1. **Protocol Compatibility:** M.2 slots are **not** universally interchangeable. An M.2 slot configured exclusively for NVMe/PCIe will not recognize an M.2 SATA drive (and vice versa).
2. **Check the Specifications:** Read your motherboard or device manual carefully (e.g., look for *"M.2_1 slot supports PCIe 4.0 x4 mode & SATA mode"*). Some slots are dual-mode, while others are NVMe-only or SATA-only.
3. **Physical Notch (Keying):**
   * **M-Key:** Standard for PCIe/NVMe drives.
   * **B+M-Key:** Standard for M.2 SATA drives (fits physically into M-slots, but still requires motherboard SATA protocol support).
4. **Form Factor (Length):** Ensure the drive length (e.g., 2280, 2242, 2230) fits your system's mounting options. 2280 is the standard size for most desktop and mini PC drives.
{{< /admonition >}}

But first, the goal was to find out if the essential components important for desktop operation work:

 * Graphics (X11) (inteldrm)
 * Network card (Wi-Fi was not so important to me, as I have CAT 7 available at the desk) (4x igc)
 * Audio (azalia)
 * USB ports

And indeed, they all worked very well. The "not configured" components are not important for operation as a desktop. WiFi can easily be replaced. The complete `dmesg` is in the box.

{{< highlight text >}}
OpenBSD 8.0-beta (GENERIC.MP) #91: Sat Aug 15 12:44:51 MDT 2026
    deraadt@amd64.openbsd.org:/usr/src/sys/arch/amd64/compile/GENERIC.MP
real mem = 34106941440 (32526MB)
avail mem = 33049096192 (31518MB)
random: good seed from bootblocks
mpath0 at root
scsibus0 at mpath0: 256 targets
mainbus0 at root
bios0 at mainbus0: SMBIOS rev. 3.6 @ 0x75c86000 (127 entries)
bios0: vendor American Megatrends International, LLC. version "0.30" date 03/26/2026
bios0: GEEKOM IX12
efi0 at bios0: UEFI 2.8
efi0: American Megatrends rev 0x5001b
acpi0 at bios0: ACPI 6.5
acpi0: sleep states S0 S5
acpi0: tables DSDT FACP FIDT MSDM SSDT SSDT SSDT SSDT SSDT HPET APIC MCFG SSDT UEFI SDEV NHLT LPIT SSDT SSDT DBGP DBG2 DMAR FPDT SSDT SSDT SSDT SSDT PHAT TPM2 WSMT
acpi0: wakeup devices PEGP(S4) PEGP(S4) PEGP(S4) SIO1(S3) RP09(S4) PXSX(S4) RP10(S4) PXSX(S4) RP11(S4) PXSX(S4) RP12(S4) PXSX(S4) RP13(S4) PXSX(S4) RP14(S4) PXSX(S4) [...]
acpitimer0 at acpi0: 3579545 Hz, 24 bits
acpihpet0 at acpi0: 19200000 Hz
acpimadt0 at acpi0 addr 0xfee00000: PC-AT compat
cpu0 at mainbus0: apid 0 (boot processor)
cpu0: Intel(R) N95, 3392.18 MHz, 06-be-00, patch 00000021
cpu0: cpuid 1 edx=bfebfbff<FPU,VME,DE,PSE,TSC,MSR,PAE,MCE,CX8,APIC,SEP,MTRR,PGE,MCA,CMOV,PAT,PSE36,CFLUSH,DS,ACPI,MMX,FXSR,SSE,SSE2,SS,HTT,TM,PBE> ecx=77fafbbf<SSE3,PCLMUL,DTES64,MWAIT,DS-CPL,VMX,EST,TM2,SSSE3,SDBG,FMA3,CX16,xTPR,PDCM,PCID,SSE4.1,SSE4.2,x2APIC,MOVBE,POPCNT,DEADLINE,AES,XSAVE,AVX,F16C,RDRAND>
cpu0: cpuid 6 eax=77<SENSOR,ARAT,PTS> ecx=9<EFFFREQ>
cpu0: cpuid 7.0 ebx=239ca7eb<FSGSBASE,TSC_ADJUST,BMI1,AVX2,SMEP,BMI2,ERMS,INVPCID,RDSEED,ADX,SMAP,CLFLUSHOPT,CLWB,PT,SHA> ecx=98c007ac<UMIP,PKU,WAITPKG,PKS> edx=fc184410<MD_CLEAR,IBT,IBRS,IBPB,STIBP,L1DF,SSBD>
cpu0: cpuid a vers=5, gp=6, gpwidth=48, ff=3, ffwidth=48
cpu0: cpuid d.1 eax=f<XSAVEOPT,XSAVEC,XGETBV1,XSAVES>
cpu0: cpuid 80000001 edx=2c100800<NXE,PAGE1GB,RDTSCP,LONG> ecx=121<LAHF,ABM,3DNOWP>
cpu0: cpuid 80000007 edx=100<ITSC>
cpu0: msr 10a=15c0fd6b<IBRS_ALL,SKIP_L1DFL,MDS_NO,IF_PSCHANGE,TAA_NO,MISC_PKG_CT,ENERGY_FILT,DOITM,SBDR_SSDP_N,FBSDP_NO,PSDP_NO,OVERCLOCK,PBRSB_NO,GDS_NO,RFDS_CLEAR>
cpu0: 32KB 64b/line 8-way D-cache, 64KB 64b/line 8-way I-cache, 2MB 64b/line 16-way L2 cache, 6MB 64b/line 12-way L3 cache
cpu0: smt 0, core 0, package 0, type E
mtrr: Pentium Pro MTRR support, 10 var ranges, 88 fixed ranges
cpu0: apic clock running at 38MHz
cpu0: mwait min=64, max=64, C-substates=0.2.0.2.0.1.0.1, IBE
cpu1 at mainbus0: apid 2 (application processor)
cpu1: Intel(R) N95, 3392.18 MHz, 06-be-00, patch 00000021
cpu1: smt 0, core 1, package 0, type E
cpu2 at mainbus0: apid 4 (application processor)
cpu2: Intel(R) N95, 2693.79 MHz, 06-be-00, patch 00000021
cpu2: smt 0, core 2, package 0, type E
cpu3 at mainbus0: apid 6 (application processor)
cpu3: Intel(R) N95, 2693.79 MHz, 06-be-00, patch 00000021
cpu3: smt 0, core 3, package 0, type E
ioapic0 at mainbus0: apid 2 pa 0xfec00000, version 20, 120 pins
acpimcfg0 at acpi0
acpimcfg0: addr 0xc0000000, bus 0-255
acpiprt0 at acpi0: bus 0 (PC00)
acpiprt1 at acpi0: bus 33 (RP09)
acpiprt2 at acpi0: bus 32 (RP10)
acpiprt3 at acpi0: bus -1 (RP11)
acpiprt4 at acpi0: bus -1 (RP12)
acpiprt5 at acpi0: bus -1 (RP13)
acpiprt6 at acpi0: bus -1 (RP14)
acpiprt7 at acpi0: bus -1 (RP15)
acpiprt8 at acpi0: bus -1 (RP16)
acpiprt9 at acpi0: bus -1 (RP01)
acpiprt10 at acpi0: bus -1 (RP02)
acpiprt11 at acpi0: bus 1 (RP03)
acpiprt12 at acpi0: bus 35 (RP04)
acpiprt13 at acpi0: bus -1 (RP05)
acpiprt14 at acpi0: bus -1 (RP06)
acpiprt15 at acpi0: bus 34 (RP07)
acpiprt16 at acpi0: bus -1 (RP08)
acpiprt17 at acpi0: bus -1 (RP17)
acpiprt18 at acpi0: bus -1 (RP18)
acpiprt19 at acpi0: bus -1 (RP19)
acpiprt20 at acpi0: bus -1 (RP20)
acpiprt21 at acpi0: bus -1 (RP21)
acpiprt22 at acpi0: bus -1 (RP22)
acpiprt23 at acpi0: bus -1 (RP23)
acpiprt24 at acpi0: bus -1 (RP24)
acpiec0 at acpi0: not present
acpipci0 at acpi0 PC00: 0x00000000 0x00000011 0x00000001
com0 at acpi0 UAR1 addr 0x3f8/0x8 irq 4: ns16550a, 16 byte fifo
"OVTI01AS" at acpi0 not configured
"OVTID858" at acpi0 not configured
"TXNW3643" at acpi0 not configured
"TXNW3643" at acpi0 not configured
"ACPI000E" at acpi0 not configured
pchgpio0 at acpi0 GPI0 addr 0xfd6e0000/0x10000 0xfd6d0000/0x10000 0xfd6a0000/0x10000 0xfd690000/0x10000 irq 14, 384 pins
acpibtn0 at acpi0: SLPB
acpicpu0 at acpi0: C3(200@1048 mwait.1@0x60), C2(350@127 mwait.1@0x21), C1(1000@1 mwait.1), PSS
acpicpu1 at acpi0: C3(200@1048 mwait.1@0x60), C2(350@127 mwait.1@0x21), C1(1000@1 mwait.1), PSS
acpicpu1: speeds: 1701, 1700, 1600, 1500, 1400, 1300, 1200, 1100, 1000, 900, 800 MHz
acpicpu2 at acpi0: C3(200@1048 mwait.1@0x60), C2(350@127 mwait.1@0x21), C1(1000@1 mwait.1), PSS
acpicpu2: speeds: 1701, 1700, 1600, 1500, 1400, 1300, 1200, 1100, 1000, 900, 800 MHz
acpicpu3 at acpi0: C3(200@1048 mwait.1@0x60), C2(350@127 mwait.1@0x21), C1(1000@1 mwait.1), PSS
acpicpu3: speeds: 1701, 1700, 1600, 1500, 1400, 1300, 1200, 1100, 1000, 900, 800 MHz
"PNP0C14" at acpi0 not configured
"PNP0C14" at acpi0 not configured
intelpmc0 at acpi0: PEPD
state 0: 0x7f:1:2:0x00:0x0000000000000060
counter: 0x7f:64:0:0x00:0x0000000000000632
frequency: 0
state 1: 0x7f:1:2:0x00:0x0000000000000060
counter: 0x00:32:0:0x03:0x00000000fe00193c
frequency: 8197
"INTC1070" at acpi0 not configured
acpibtn1 at acpi0: PWRB
tpm0 at acpi0 TPM_ 2.0 (TIS) addr 0xfed40000/0x5000, device 0x001d15d1 rev 0x36
acpipwrres0 at acpi0: WRST
acpipwrres1 at acpi0: TBT0, resource for TDM0, TRP0, TRP1
acpipwrres2 at acpi0: TBT1, resource for TDM1, TRP2, TRP3
acpipwrres3 at acpi0: D3C_, resource for TXHC, TDM0, TDM1, TRP0, TRP1, TRP2, TRP3
acpipwrres4 at acpi0: FN00
acpipwrres5 at acpi0: FN01
acpipwrres6 at acpi0: FN02
acpipwrres7 at acpi0: FN03
acpipwrres8 at acpi0: FN04
acpitz0 at acpi0
acpitz0: critical temperature is 110 degC
acpipwrres9 at acpi0: PIN_
acpivideo0 at acpi0: GFX0
acpivout0 at acpivideo0: DD1F
acpivout1 at acpivideo0: DD2F
cpu0: using VERW MDS workaround
cpu0: Enhanced SpeedStep 3392 MHz: speeds: 1701, 1700, 1600, 1500, 1400, 1300, 1200, 1100, 1000, 900, 800 MHz
pci0 at mainbus0 bus 0
0:31:5: mem address conflict 0xfe010000/0x1000
pchb0 at pci0 dev 0 function 0 vendor "Intel", unknown product 0x4618 rev 0x00
inteldrm0 at pci0 dev 2 function 0 "Intel Graphics" rev 0x00
drm0 at inteldrm0
inteldrm0: msi, ALDERLAKE_P, gen 12
"Intel ADL-N GNA" rev 0x00 at pci0 dev 8 function 0 not configured
"Intel Core 12G CL" rev 0x01 at pci0 dev 10 function 0 not configured
xhci0 at pci0 dev 13 function 0 "Intel ADL-N xHCI" rev 0x00: msi, xHCI 1.20
usb0 at xhci0: USB revision 3.0
uhub0 at usb0 configuration 1 interface 0 "Intel xHCI root hub" rev 3.00/1.00 addr 1
xhci1 at pci0 dev 20 function 0 "Intel ADL-N xHCI" rev 0x00: msi, xHCI 1.20
usb1 at xhci1: USB revision 3.0
uhub1 at usb1 configuration 1 interface 0 "Intel xHCI root hub" rev 3.00/1.00 addr 1
"Intel ADL-N SRAM" rev 0x00 at pci0 dev 20 function 2 not configured
dwiic0 at pci0 dev 21 function 0 "Intel ADL-N I2C" rev 0x00: apic 2 int 27
iic0 at dwiic0
dwiic1 at pci0 dev 21 function 1 "Intel ADL-N I2C" rev 0x00: apic 2 int 40
iic1 at dwiic1
"Intel ADL-N HECI" rev 0x00 at pci0 dev 22 function 0 not configured
ahci0 at pci0 dev 23 function 0 "Intel ADL-N AHCI" rev 0x00: msi, AHCI 1.3.1
ahci0: port 0: 6.0Gb/s
ahci0: PHY offline on port 1
scsibus1 at ahci0: 32 targets
sd0 at scsibus1 targ 0 lun 0: <ATA, Samsung SSD 860, RVT2> naa.5002538e3130b347
sd0: 476940MB, 512 bytes/sector, 976773168 sectors, thin
dwiic2 at pci0 dev 25 function 0 "Intel ADL-N I2C" rev 0x00: apic 2 int 31
iic2 at dwiic2
dwiic3 at pci0 dev 25 function 1 "Intel ADL-N I2C" rev 0x00: apic 2 int 32
iic3 at dwiic3
sdhc0 at pci0 dev 26 function 0 "Intel ADL-N eMMC" rev 0x00: apic 2 int 16
sdhc0: SDHC 3.00, 200 MHz base clock
sdmmc0 at sdhc0: 8-bit, sd high-speed, mmc high-speed, ddr52, dma
ppb0 at pci0 dev 28 function 0 "Intel ADL-N PCIE" rev 0x00: msi
pci1 at ppb0 bus 1
"MediaTek MT7920" rev 0x00 at pci1 dev 0 function 0 not configured
ppb1 at pci0 dev 28 function 3 "Intel ADL-N PCIE" rev 0x00: msi
pci2 at ppb1 bus 35
igc0 at pci2 dev 0 function 0 "Intel I226-V" rev 0x04, msix, 4 queues, address 38:f7:cd:d3:df:90
ppb2 at pci0 dev 28 function 6 "Intel ADL-N PCIE" rev 0x00: msi
pci3 at ppb2 bus 34
igc1 at pci3 dev 0 function 0 "Intel I226-V" rev 0x04, msix, 4 queues, address 38:f7:cd:d9:13:e2
ppb3 at pci0 dev 29 function 0 "Intel ADL-N PCIE" rev 0x00: msi
pci4 at ppb3 bus 33
igc2 at pci4 dev 0 function 0 "Intel I226-V" rev 0x04, msix, 4 queues, address 38:f7:cd:d9:13:dc
ppb4 at pci0 dev 29 function 1 "Intel ADL-N PCIE" rev 0x00: msi
pci5 at ppb4 bus 32
igc3 at pci5 dev 0 function 0 "Intel I226-V" rev 0x04, msix, 4 queues, address 38:f7:cd:d9:13:dd
"Intel ADL-N UART" rev 0x00 at pci0 dev 30 function 0 not configured
"Intel ADL-N GSPI" rev 0x00 at pci0 dev 30 function 3 not configured
pcib0 at pci0 dev 31 function 0 vendor "Intel", unknown product 0x5482 rev 0x00
azalia0 at pci0 dev 31 function 3 "Intel ADL-N HD Audio" rev 0x00: msi
azalia0: codecs: Realtek ALC269
audio0 at azalia0
ichiic0 at pci0 dev 31 function 4 "Intel ADL-N SMBus" rev 0x00: apic 2 int 16
iic4 at ichiic0
iic4: addr 0x4a 15=2c 16=20 19=04 1b=05 1c=60 1e=60 1f=60 20=89 21=78 22=63 25=78 26=63 27=78 28=63 29=b0 2a=bb 2b=42 2c=20 2d=22 2e=04 2f=5a 32=80 34=0e 3b=04 3c=0b 3d=2a words 00=0000 01=0000 02=0000 03=0000 04=0000 05=0000 06=0000 07=0000
"eeprom" at iic4 addr 0x52 not configured
"Intel ADL-N SPI" rev 0x00 at pci0 dev 31 function 5 not configured
isa0 at pcib0
isadma0 at isa0
pcppi0 at isa0 port 0x61
spkr0 at pcppi0
vmm0 at mainbus0: VMX/EPT
efifb at mainbus0 not configured
enable mbufs in high memory
scsibus2 at sdmmc0: 2 targets, initiator 0
sd1 at scsibus2 targ 1 lun 0: <SD/MMC, C9A611, 0000> removable
sd1: 59680MB, 512 bytes/sector, 122224640 sectors
uhidev0 at uhub1 port 1 configuration 1 interface 0 "CHESEN PS2 to USB Converter" rev 1.10/0.10 addr 2
uhidev0: iclass 3/1
ukbd0 at uhidev0: 8 variable keys, 6 key codes
wskbd0 at ukbd0: console keyboard
uhidev1 at uhub1 port 1 configuration 1 interface 1 "CHESEN PS2 to USB Converter" rev 1.10/0.10 addr 2
uhidev1: iclass 3/1, 3 report ids
ums0 at uhidev1 reportid 1: 5 buttons, Z dir
wsmouse0 at ums0 mux 0
uhid0 at uhidev1 reportid 2: input=1, output=0, feature=0
ucc0 at uhidev1 reportid 3: 5 usages, 4 keys, array
wskbd1 at ucc0 mux 1
uhidev2 at uhub1 port 3 configuration 1 interface 0 "YICHIP Wireless Device" rev 2.00/0.02 addr 3
uhidev2: iclass 3/1
ukbd1 at uhidev2: 8 variable keys, 6 key codes
wskbd2 at ukbd1 mux 1
uhidev3 at uhub1 port 3 configuration 1 interface 1 "YICHIP Wireless Device" rev 2.00/0.02 addr 3
uhidev3: iclass 3/1, 3 report ids
ums1 at uhidev3 reportid 1: 5 buttons, Z and W dir
wsmouse1 at ums1 mux 0
uhid1 at uhidev3 reportid 2: input=1, output=0, feature=0
ucc1 at uhidev3 reportid 3: 897 usages, 20 keys, array
wskbd3 at ucc1 mux 1
ugen0 at uhub1 port 8 "MediaTek Inc. Wireless_Device" rev 2.10/1.00 addr 4
vscsi0 at root
scsibus3 at vscsi0: 256 targets
softraid0 at root
scsibus4 at softraid0: 256 targets
root on sd0a (ba0a2c4309ee65cb.a) swap on sd0b dump on sd0b
inteldrm0: 3440x1440, 32bpp
wsdisplay0 at inteldrm0 mux 1: console (std, vt100 emulation), using wskbd0
wskbd1: connecting to wsdisplay0
wskbd2: connecting to wsdisplay0
wskbd3: connecting to wsdisplay0
wsdisplay0: screen 1-5 added (std, vt100 emulation)
{{< /highlight >}}

Under the hood of the Geekom IX12 works an Intel N95 processor, which is based on Intel's energy-efficient [Alder Lake N](https://en.wikipedia.org/wiki/Alder_Lake) architecture. This system-on-a-chip relies exclusively on four efficient Gracemont cores ("E-Cores"), dispenses with power-hungry performance cores, and with a maximum thermal design power (TDP) of just a few watts, is perfectly suited for fanless mini-PCs.

---

## The Frequency Riddle & the Mailing List Solution

When first testing with `sysctl hw.cpuspeed`, one value startled me: it remained stubbornly stuck at **1701 MHz**. Neither `sysctl hw.perfpolicy=high` nor synthetic benchmarks changed this number. Even going into the BIOS to deactivate *Intel Speed Shift* or to enable Turbo Mode seemed, at first glance, to change nothing in the display.

An inquiry on the OpenBSD mailing list finally brought crucial light to the matter: `hw.cpuspeed` merely reads out the primary ACPI tables. The 1701 embodies the Turbo Mode. However, the real, current live frequencies of the cores can be precisely observed via **`sysctl hw.sensors`**!

{{< admonition type="tip" title="Community Insight" open=true >}}
OpenBSD pushes the turbo frequency dynamically up to the maximum – here 3.2GHz – during load peaks. Under heavy multi-core continuous load, the BIOS power limits (PL1/PL2) kick in to protect the passive cooling, and the system settles stably at 2.7GHz.
{{< /admonition >}}

The measurement series with `sysctl hw.sensors` confirm this perfect behavior:

* **Idle:** A dreamlike 38 °C at a minimal **800 MHz** on all cores.
* **Short Burst (4x sha256 -tt):** The CPU immediately shoots up briefly to **3.35 GHz** (at approx. 54 °C).
* **Continuous Load (4x sha256 -tt):** After the sustained power limit (PL1) takes hold, all 4 cores settle stably at **2.70 GHz** and a cool 49 °C.
* **Partial Load (2x sha256 -tt):** Since the 15W budget is distributed over only two cores, the frequencies remain permanently high between **3.00 GHz and 3.25 GHz** (at approx. 62 °C).

The frequency management thus works absolutely "as designed" and extracts the optimum from the four efficient cores of the Intel N95.

---

## Performance Test: Compiling Under OpenBSD

Riddles resolved – how do the two computers stack up in a direct comparison?

CPU-intensive activities are won by the old HP: 4x 3.2 GHz continuous power under full load is simply more in terms of pure computing power than 4x 2.7 GHz all-core clock speed. With mixed workloads (CPU, memory, and I/O), however, the Geekom performs extremely respectably and is even noticeably faster with a high I/O component!

To create fair conditions, both machines ran with identical specifications (32 GB RAM, `hw.smt=1`, `apm -H`, `noatime` mount options).

### 1. OpenJFX (incl. WebKit)

Compiling OpenJFX represents a gigantic, continuous CPU load.

| System / Setup | `make` (Real Time) | `make fake` (Real Time) |
| :--- | :--- | :--- |
| **HP Compaq Pro** (SSD) | **172m 11.28s** | 0m 02.20s |
| **Geekom IX12** (Samsung SSD) | **178m 15.70s** | 0m 01.19s |
| **Geekom IX12** (Verbatim SSD) | **182m 29.37s** | 0m 01.32s |

Here, the old HP is ahead by just under 6 minutes, as it can keep cooling its 3.2 GHz without restriction. However, the Geekom keeps up bravely with its 2.7 GHz all-core clock speed. But what are 6 minutes difference when it takes almost 3 hours?

### 2. Autopsy (`make clean && make fake`)

A mixed build process with plenty of directory structure, script, and disk I/O:

| System / Setup | Real Time | User Time | System Time |
| :--- | :--- | :--- | :--- |
| **HP Compaq Pro** | 8m 35.53s | 6m 45.57s | 2m 10.93s |
| **Geekom IX12** (Samsung SSD) | **6m 08.76s** | 6m 44.93s | 1m 19.31s |
| **Geekom IX12** (Verbatim SSD) | **6m 37.05s** | 6m 34.27s | 1m 21.61s |

Here, the [Geekom IX12](https://www.awin1.com/cread.php?awinmid=32589&awinaffid=3025379&ued=https%3A%2F%2Fwww.geekom.de%2Fgeekom-ix12-luefterloser-mini-pc%2F#affiliate) wins hands down: it beats the old HP by **almost 2.5 minutes**. The more modern platform, faster memory, newer bus architecture, and the Samsung SSD (which itself shaves just under 30 seconds off the Verbatim's time) ensure a decent I/O advantage.

---

## Conclusion & Outlook

The switch from the old HP Compaq Pro to the [Geekom IX12](https://www.awin1.com/cread.php?awinmid=32589&awinaffid=3025379&ued=https%3A%2F%2Fwww.geekom.de%2Fgeekom-ix12-luefterloser-mini-pc%2F#affiliate) was a complete success! As a replacement for the DVD drive, just in case, I recently picked up a mobile DVD drive as a 3 EUR bargain from a flea market ;)

* **Silent:** Absolute silence at the desk – no more continuous fan noise!
* **Efficient:** Power consumption is vanishingly small compared to the old Ivy Bridge platform.
* **Snappy in Daily Use:** For everyday tasks, I/O-heavy tasks, and short bursts, the Geekom even feels noticeably more responsive due to the higher single-core turbo.

Small drawback: A few more USB ports would have been nice; for now, the monitor has to serve as a USB hub.

Although the Geekom is primarily advertised as an industrial PC (optimal as a firewall or router), with a bit more RAM and a larger SSD, the device also makes an excellent figure as a desktop! For the future, the setup remains an ideal base – whether it continues as a noiseless daily-driver workstation or later as a high-performance 4x 2.5G OpenBSD router in the home lab. The hardware has definitely proven itself in the test arena!

