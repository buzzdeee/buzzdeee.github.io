---
title: "Fujitsu Futro S920 und OpenBSD"
subtitle: "Erfahrungsbericht zu Hardware, Kompatibilität und Einsatzmöglichkeiten"
date: 2026-09-02T22:46:03+02:00
lastmod: 2026-09-02T22:46:03+02:00
draft: false
description: "Ein sachlicher Bericht über den jahrelangen Einsatz des Fujitsu Futro S920 Thin Clients mit OpenBSD."
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
  images: [ "/images/Fujitsu_Futro_S920_featured.jpg", "/images/futro-s920.jpg", "/images/futro-s920-open.jpg" ]
  # ...
---

Über einige Jahre hinweg liefen bei mir einige Fujitsu Futro S920 als verlässliche, lautlose Server für verschiedene Netzwerkdienste im lokalen Netz. Mittlerweile habe ich die Infrastruktur schrittweise erneuert und die alten Instanzen durch neuere Futro S9010 ersetzt. 

Bevor die ausgemusterten S920-Geräte demnächst auf [eBay](https://ebay.us/nn1zUq) den Besitzer wechseln, folgt hier eine sachliche Zusammenfassung zu Hardware, Stromverbrauch und OpenBSD-Kompatibilität. Auch wenn die Architektur nicht mehr ganz taufrisch ist, leisten diese Thin Clients als stromsparende Nodes nach wie vor gute Dienste.

<figure class="ma0 w-75">
  <img src="/images/futro-s920.jpg" alt="Fujitsu Futro S920 Front- und Rückansicht">
  <figcaption>Der Fujitsu Futro S920 in Front- und Rückansicht.</figcaption>
</figure>

---

## Hardware-Eigenschaften und Schnittstellen

Der Futro S920 setzt auf ein komplett passiv gekühltes Design ohne bewegliche Teile. Das Gehäuse ist leicht zugänglich, was Wartung oder Komponentenwechsel unkompliziert macht.

{{< admonition type="info" title="Hardware-Übersicht & Ausbau" >}}
* **Prozessor:** AMD GX-222GC SOC (2 Kerne, 2.2 GHz, Turbo bis 2.4 GHz) mit Radeon R5E Grafik
* **Arbeitsspeicher (RAM):** 2x DDR3 SO-DIMM Slots. Laut Fujitsu-Datenblatt werden offiziell bis zu 8 GB unterstützt. In der Praxis lassen sich je nach Modulbestückung und BIOS-Stand oft bis zu 16 GB (2x 8 GB SO-DIMM) addressieren.
* **M.2 / mSATA:** 1x mSATA-Steckplatz (ideal für eine kompakte System-SSD zum Booten des Betriebssystems)
* **SATA-Anschluss:** 1x nativer 2,5″-SATA-Port inkl. Stromanschluss auf der Platine (für größere SSDs oder HDDs als Datenspeicher)
* **Erweiterbarkeit:** 1x PCIe-Steckplatz (erfordert eine interne Riser-Karte)
* **Serielle Ports:** 2x native RS-232-Schnittstellen (`com0` und `com1`) auf der Rückseite
* **Netzwerk:** 1x Gigabit Ethernet (Realtek RTL8168/8111G)
* **Netzteil & Verbrauch:** Ausgelegt für externe 19V-Notebook-Netzteile. Die Leistungsaufnahme liegt im Leerlauf (Idle) unter OpenBSD typischerweise bei sparsamen **7 bis 9 Watt**. Unter voller Syntheselast steigt der Verbrauch selten über **15 bis 18 Watt**.
* **Spezifikationen & Handbuch:** [Fujitsu FUTRO S920 Operating Manual](https://support.ts.fujitsu.com/Search/SWP1206702.asp)
{{< /admonition >}}

Die Trennung zwischen dem mSATA-Steckplatz (für die OpenBSD-Systeminstallation) und dem 2,5″-SATA-Port (für Datenbestände) macht die Plattform erstaunlich flexibel für Storage- oder Netzwerkaufgaben.


<figure class="ma0 w-75">
  <img src="/images/futro-s920-open.jpg" alt="Fujitsu Futro S920 Innenansicht">
  <figcaption>Der Fujitsu Futro S920 Innenansicht.</figcaption>
</figure>

---

## Hardware-Erkennung unter OpenBSD

Sämtliche onboard integrierten Chipsätze werden vom OpenBSD-Kernel direkt unterstützt. Weder für die Netzwerkhardware noch für Grafik oder Audio sind zusätzliche Verrenkungen notwendig.

Der folgende Auszug zeigt den originalen Boot-Log (`dmesg`) unter OpenBSD 8.0-beta:

{{< highlight text >}}
OpenBSD 8.0-beta (GENERIC.MP) #2: Mon Aug 31 11:45:08 MDT 2026
    deraadt@amd64.openbsd.org:/usr/src/sys/arch/amd64/compile/GENERIC.MP
real mem = 3918221312 (3736MB)
avail mem = 3777544192 (3602MB)
random: good seed from bootblocks
mpath0 at root
scsibus0 at mpath0: 256 targets
mainbus0 at root
bios0 at mainbus0: SMBIOS rev. 2.8 @ 0xacec9018 (55 entries)
bios0: vendor FUJITSU // American Megatrends Inc. version "V4.6.5.4 R1.16.0 for D3313-G1x" date 08/13/2018
bios0: FUJITSU FUTRO S920
acpi0 at bios0: ACPI 5.0
acpi0: sleep states S0 S3 S4 S5
acpi0: tables DSDT FACP APIC FPDT FIDT TCPA MCFG HPET SSDT SSDT SSDT SSDT SSDT
acpi0: wakeup devices LAN1(S4) LAN2(S4) LAN3(S4) SBAZ(S4) EHC1(S4) EHC2(S4) EHC3(S4) XHC0(S4) GFX_(S3)
acpitimer0 at acpi0: 3579545 Hz, 32 bits
acpimadt0 at acpi0 addr 0xfee00000: PC-AT compat
cpu0 at mainbus0: apid 0 (boot processor)
cpu0: AMD GX-222GC SOC with Radeon(TM) R5E Graphics, 2195.97 MHz, 16-30-01, patch 07030106
cpu0: cpuid 1 edx=178bfbff<FPU,VME,DE,PSE,TSC,MSR,PAE,MCE,CX8,APIC,SEP,MTRR,PGE,MCA,CMOV,PAT,PSE36,CFLUSH,MMX,FXSR,SSE,SSE2,HTT> ecx=76d8220b<SSE3,PCLMUL,MWAIT,SSSE3,CX16,SSE4.1,SSE4.2,MOVBE,POPCNT,AES,XSAVE,AVX,F16C,RDRAND>
cpu0: cpuid 6 eax=4<ARAT> ecx=1<EFFFREQ>
cpu0: cpuid 7.0 ebx=8<BMI1>
cpu0: cpuid d.1 eax=1<XSAVEOPT>
cpu0: cpuid 80000001 edx=2fd3fbff<NXE,MMXX,FFXSR,PAGE1GB,RDTSCP,LONG> ecx=1d4037ff<LAHF,CMPLEG,SVM,EAPICSP,AMCR8,ABM,SSE4A,MASSE,3DNOWP,OSVW,IBS,SKINIT,TOPEXT,DBKP,PERFTSC,PCTRL3>
cpu0: cpuid 80000007 edx=33d9<HWPSTATE,ITSC>
cpu0: cpuid 80000008 ebx=1000<IBPB>
cpu0: 32KB 64b/line 8-way D-cache, 32KB 64b/line 2-way I-cache, 1MB 64b/line 16-way L2 cache
cpu0: smt 0, core 0, package 0, type P
mtrr: Pentium Pro MTRR support, 8 var ranges, 88 fixed ranges
cpu0: apic clock running at 99MHz
cpu0: mwait min=64, max=64, IBE
cpu1 at mainbus0: apid 1 (application processor)
cpu1: AMD GX-222GC SOC with Radeon(TM) R5E Graphics, 2196.08 MHz, 16-30-01, patch 07030106
cpu1: smt 0, core 1, package 0, type P
ioapic0 at mainbus0: apid 3 pa 0xfec00000, version 21, 24 pins
ioapic1 at mainbus0: apid 4 pa 0xfec01000, version 21, 32 pins
acpimcfg0 at acpi0
acpimcfg0: addr 0xe0000000, bus 0-255
acpihpet0 at acpi0: 14318180 Hz
acpiprt0 at acpi0: bus 0 (PCI0)
acpiprt1 at acpi0: bus 1 (GPP0)
acpiprt2 at acpi0: bus -1 (GPP1)
acpiprt3 at acpi0: bus -1 (GPP2)
acpiprt4 at acpi0: bus -1 (GPP3)
acpiprt5 at acpi0: bus -1 (GFX_)
acpipci0 at acpi0 PCI0: 0x00000000 0x00000011 0x00000001
com0 at acpi0 UAR0 addr 0x3f8/0x8 irq 4: ns16550a, 16 byte fifo
acpicmos0 at acpi0
"PNP0303" at acpi0 not configured
com1 at acpi0 UAR1 addr 0x2f8/0x8 irq 3: ns16550a, 16 byte fifo
"FUJ02E3" at acpi0 not configured
acpibtn0 at acpi0: PWRB
tpm0 at acpi0 TPM_ 1.2 (TIS) addr 0xfed40000/0x5000, device 0x001a15d1 rev 0x10
acpicpu0 at acpi0: C2(0@400 io@0x414), C1(@1 halt!), PSS
acpicpu1 at acpi0: C2(0@400 io@0x414), C1(@1 halt!), PSS
acpivideo0 at acpi0: VGA_
acpivout0 at acpivideo0: LCD_
acpivideo1 at acpi0: VGA_
cpu0: 2195 MHz: speeds: 2200 2000 1800 1600 1400 1200 1000 MHz
pci0 at mainbus0 bus 0
pchb0 at pci0 dev 0 function 0 "AMD 16h Root Complex" rev 0x00
radeondrm0 at pci0 dev 1 function 0 "ATI Mullins" rev 0x06
drm0 at radeondrm0
radeondrm0: msi
azalia0 at pci0 dev 1 function 1 "ATI Radeon HD Audio" rev 0x00: msi
azalia0: no supported codecs
pchb1 at pci0 dev 2 function 0 "AMD 16h Host" rev 0x00
ppb0 at pci0 dev 2 function 2 "AMD 16h PCIE" rev 0x00: msi
pci1 at ppb0 bus 1
re0 at pci1 dev 0 function 0 "Realtek 8168" rev 0x0c: RTL8168G/8111G (0x4c00), msi, address 4c:52:62:08:a6:d3
rgephy0 at re0 phy 7: RTL8251, rev. 0
ccp0 at pci0 dev 8 function 0 "AMD 16h Crypto" rev 0x00
xhci0 at pci0 dev 16 function 0 "AMD Bolton xHCI" rev 0x11: msix, xHCI 1.0
usb0 at xhci0: USB revision 3.0
uhub0 at usb0 configuration 1 interface 0 "AMD xHCI root hub" rev 3.00/1.00 addr 1
ahci0 at pci0 dev 17 function 0 "AMD Hudson-2 SATA" rev 0x40: msi, AHCI 1.3
ahci0: port 1: 6.0Gb/s
scsibus1 at ahci0: 32 targets
sd0 at scsibus1 targ 1 lun 0: <ATA, SanDisk SSD U100, 10.5> naa.5001b44a174a11ed
sd0: 30533MB, 512 bytes/sector, 62533296 sectors, thin
ehci0 at pci0 dev 18 function 0 "AMD Hudson-2 USB2" rev 0x39: apic 3 int 18
usb1 at ehci0: USB revision 2.0
uhub1 at usb1 configuration 1 interface 0 "AMD EHCI root hub" rev 2.00/1.00 addr 1
ehci1 at pci0 dev 19 function 0 "AMD Hudson-2 USB2" rev 0x39: apic 3 int 18
usb2 at ehci1: USB revision 2.0
uhub2 at usb2 configuration 1 interface 0 "AMD EHCI root hub" rev 2.00/1.00 addr 1
piixpm0 at pci0 dev 20 function 0 "AMD Hudson-2 SMBus" rev 0x42: SMI
iic0 at piixpm0
spdmem0 at iic0 addr 0x51: 4GB DDR3 SDRAM PC3-12800 SO-DIMM
iic1 at piixpm0
azalia1 at pci0 dev 20 function 2 "AMD Hudson-2 HD Audio" rev 0x02: apic 3 int 16
azalia1: codecs: Realtek ALC671
audio0 at azalia1
pcib0 at pci0 dev 20 function 3 "AMD Hudson-2 LPC" rev 0x11
pchb2 at pci0 dev 24 function 0 "AMD 16h Link Cfg" rev 0x00
pchb3 at pci0 dev 24 function 1 "AMD 16h Address Map" rev 0x00
pchb4 at pci0 dev 24 function 2 "AMD 16h DRAM Cfg" rev 0x00
km0 at pci0 dev 24 function 3 "AMD 16h Misc Cfg" rev 0x00
pchb5 at pci0 dev 24 function 4 "AMD 16h CPU Power" rev 0x00
pchb6 at pci0 dev 24 function 5 "AMD 16h Misc Cfg" rev 0x00
isa0 at pcib0
isadma0 at isa0
pckbc0 at isa0 port 0x60/5 irq 1 irq 12
pckbd0 at pckbc0 (kbd slot)
wskbd0 at pckbd0: console keyboard
pcppi0 at isa0 port 0x61
spkr0 at pcppi0
vmm0 at mainbus0: SVM/RVI
enable mbufs in high memory
uhub3 at uhub1 port 1 configuration 1 interface 0 "Advanced Micro Devices Hub" rev 2.00/0.18 addr 2
uhub4 at uhub2 port 1 configuration 1 interface 0 "Advanced Micro Devices Hub" rev 2.00/0.18 addr 2
vscsi0 at root
scsibus2 at vscsi0: 256 targets
softraid0 at root
scsibus3 at softraid0: 256 targets
root on sd0a (125adc9b29c30570.a) swap on sd0b dump on sd0b
radeondrm0: MULLINS
radeondrm0: 1920x1080, 32bpp
wsdisplay0 at radeondrm0 mux 1: console (std, vt100 emulation), using wskbd0
wsdisplay0: screen 1-5 added (std, vt100 emulation)
{{< /highlight >}}

{{< admonition type="note" title="Grafikanbindung & Xorg" >}}
Das Modul `radeondrm0` bindet die Grafikkarte ("AMD Mullins") nativ ein. Sowohl das Terminal (`wsdisplay0`) in voller Auflösung (1920x1080) als auch ein optionaler X11-Server laufen ohne Nachbearbeitung der Treiber.
{{< /admonition >}}

{{< admonition type="warning" title="Wichtiger Hinweis: DRM-Warnungen im dmesg beheben" >}}
Sollten beim Systemstart DRM-Warnmeldungen im `dmesg` auftreten, wie etwa:

{{< highlight text >}}
drm:pid0:drm_fb_helper_find_format *WARNING* [drm] format 0x36314752 not supported
drm:pid0:drm_fb_helper_find_format *WARNING* [drm] format 0x36314752 not supported
drm:pid0:__drm_fb_helper_find_sizes *WARNING* [drm] No compatible format found
{{< /highlight >}}

liegt die Ursache an einem zu knapp bemessenen Grafikspeicher im BIOS.

**Lösung:** Beim Systemstart ins BIOS wechseln und das **UMA Framebuffer Memory auf mindestens 256MB** anheben. Danach lädt `radeondrm0` sauber und das System läuft ohne Fehlermeldungen auf voller Display-Auflösung.
{{< /admonition >}}

---

## Einsatzgebiete im Rückblick

Aufgrund der Leistungsdaten und des geringen Stromverbrauchs deckte der S920 bei mir folgende Aufgaben ab:

1. **Infrastruktur-Dienste:** Ausführung von `nsd` und `unbound` (lokaler DNS-Resolver), `dhcpd` sowie Bereitstellung von PXE/TFTP-Boot-Images via `tftpd` für Neuinstallationen im Netzwerk.
2. **Konsolensystem:** Verwaltung externer Geräte über die beiden seriellen Anschlüsse (`com0`/`com1`).
3. **Kiosk- oder Arbeitsplatzsystem:** Gelegentlicher Betrieb mit grafischer Oberfläche für schlichte Monitoring-Anzeigen.

---

## Fazit & Ausblick

Der Wechsel auf den Futro S9010 bringt vor allem mehr Rechenleistung, neuere CPU-Instruktionen und eine modernere Plattform mit. Dennoch bleibt der Futro S920 eine brauchbare, geräuschlose Hardware für minimalistische Setups, OpenBSD-Einstiege oder einfache Netzwerk-Server. 

Wer gebraucht nach einer günstigen Basis mit nativen seriellen Ports und geringem Strombedarf sucht, wird man schnell bei [eBay](https://ebay.us/UlXc9q) fündig, mit unterschiedlicher Ausstattung. Man erhält mit dem S920 ein solides Stück Hardware zum kleinen Preis. 

