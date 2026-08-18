---
title: "OpenBSD Upgrades: Mein Wechsel vom HP Compaq Pro zum lautlosen Geekom IX12"
subtitle: "Weniger Strom, null Lärm und Performance-Erkenntnisse im Alltag."
date: 2026-08-17T16:50:10+02:00
lastmod: 2026-08-17T16:50:10+02:00
draft: false
author: ""
authorLink: ""
description: "Ein Erfahrungsbericht über den Umstieg von einem betagten HP Compaq 6300 Desktop auf einen lautlosen Geekom IX12 Mini-PC unter OpenBSD inkl. Benchmark-Vergleichen beim Kompilieren."
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
  images: [ "/images/hp_vs_geekom.jpg", "/images/geekom_unboxed.jpeg", "/images/geekom_front.jpeg", "/images/geekom_back.jpeg", "/images/geekom_open.jp
eg", "/images/geekom_open_no_disk.jpeg" ]
  # ...
---

Ein verlässliches Arbeitstier ist etwas Feines. Über Jahre hinweg leistete mir ein betagter Desktop-Knecht treue Dienste für das tägliche Internet-Browsing, kleine Admin-Aufgaben und das gelegentliche Kompilieren von Ports unter OpenBSD. Es handelt sich um einen **HP Compaq Pro 6300 SFF** (hergestellt um das Jahr **2012**). 

{{< admonition type="note" title="Der alte Weggefährte: HP Compaq Pro 6300 SFF" open=false >}}
Der HP Compaq Pro 6300 SFF war um 2012 ein typischer Business-Workhorse-PC. Bestückt mit einem Intel Core i5-3470 (Ivy Bridge, 4 Kerne, 3.20 GHz) und 32 GB DDR3 RAM hat er zwar auch nach über einer Dekade noch klaglos seinen Dienst verrichtet, glänzte aber naturgemäß weder bei der Energieeffizienz noch bei der Geräuschentwicklung.
{{< /admonition >}}

Hier ist der vollständige Kernel-Log (`dmesg`) des alten HP-Systems zum Nachlesen:

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

## Der Neue im Office

Ein Ersatz musste her! Die Zielsetzung war klar umrissen: **Fanless (passiv gekühlt), stromsparend und ausreichend stark für den OpenBSD-Alltag.** Nach einigem Suchen fiel die Wahl auf den **[Geekom IX12](https://www.awin1.com/cread.php?awinmid=32589&awinaffid=3025379&ued=https%3A%2F%2Fwww.geekom.de%2Fgeekom-ix12-luefterloser-mini-pc%2F)**. Ich habe über die letzten Jahre diverse Rechner und Switches ausgetauscht, das letzte verbliebene Gerät mit Lüfter, das regelmäßig eingeschaltet ist, ist mein Desktop. Ein "lüfterloses" Büro ist eine feine Sache, frage mich wie ich das früher ausgehalten habe, zeitweise auch mit Rechnern mit kaputten Lüftern, die ganz schön genervt hatten. Vielleicht wird man aber auch einfach nur älter ;)

---

## Die Hardware unter der Lupe

### Unboxing des Geekom IX12

Schon beim ersten Auspacken überzeugt der Geekom IX12 mit einem hochwertigen, eloxierten Gehäuse. Seine massiven Kühlrippen leiten die Abwärme des Intel N95 komplett lüfterlos ab. Werksseitig kommt das Gerät mit 8 GB RAM und einer 256 GB M.2 SATA-SSD.  Nicht unglaublich üppig, aber für eine Firewall/Router, oder Netzwerkserver vollkommen ausreichend. Schließlich wird er ja auch als "Die ultimative Firewall & Gateway-Lösung" bei Geekom angepriesen.

Der Lieferumfang fällt üppig aus: Ein solides Netzteil mit Adaptern, diverse Kabel, eine VESA-Halterung sowie externe Antennen liegen bei. Die Anschlüsse bieten alles, was das Herz begehrt: gleich **vier 2.5G-Netzwerkports**, ein USB-C Console Port, fünf weitere USB-Anschlüsse sowie je ein HDMI- und DisplayPort-Ausgang.

{{< figure src="/images/geekom_unboxed.jpeg" alt="Geekom IX12 Lieferumfang und Anschlüsse" caption="Der Geekom IX12 glänzt mit eloxiertem Gehäuse, Kühlrippen und reichlich Zubehör." class="ma0 w-75" >}}

{{< admonition type="note" title="Perfekt fürs Home-Lab" open=true >}}
Besonders die Kombination aus vier vollwertigen 2.5-Gigabit-Netzwerkanschlüssen, Konsolen Port und der passiven Kühlung macht das Gerät zu einer fantastischen Plattform für Router, OpenBSD-Firewalls oder Edge-Knoten. Vorinstalliert ist ab Werk allerdings Windows 11 – was natürlich zügig durch OpenBSD ersetzt wurde! ;)
{{< /admonition >}}

### Hardware außen

Äußerlich macht er einen sehr soliden Eindruck, stabiles Gehäuse, Anschlüsse vorn und hinten. 

{{< figure src="/images/geekom_front.jpeg" alt="Geekom IX12 Mini PC Vorn" caption="Der Geekom IX12 Frontansicht – USB, Display, SIM-Karte, Audio und Power Button." class="ma0 w-75" >}}
{{< figure src="/images/geekom_back.jpeg" alt="Geekom IX12 Mini PC Hinten" caption="Der Geekom IX12 Rückansicht – 4x LAN, Konsole, Stromanschluss." class="ma0 w-75" >}}

### Hardware innen

Man drehe das Gerät auf den Kopf, entferne die 4 Schräubchen, und schon lässt sich der Deckel entfernen, und man kommt sehr einfach an alle wechselbaren Teile ran. RAM, SATA M.2 SSD, WiFi, etc. nichts verlötet, sondern einfach erreichbar und austauschbar, wie man es sich wünscht.

{{< figure src="/images/geekom_open.jpeg" alt="Geekom IX12 Mini geöffnet" caption="Der Geekom IX12 geöffnet." class="ma0 w-75" >}}
{{< figure src="/images/geekom_open_no_disk.jpeg" alt="Geekom IX12 geöffnet, RAM und Disk entfernt" caption="Geekom IX12 geöffnet, RAM und Disk entfernt." class="ma0 w-75" >}}

### Hardware Upgrades

Für ein vollwertiges System als Desktop und zum Kompilieren von Ports durfte es etwas mehr sein. Daher bekam der Mini-PC ein Upgrade:

* **Arbeitsspeicher:** Aufgerüstet von 8 GB auf **[32 GB RAM](https://ebay.us/fmXn5I)**
* **Speicher:** Ersetzung der 256 GB SSD durch ein **512 GB Modell**

Um den Einfluss des Speichermediums zu prüfen, kamen zwei M.2 SATA SSDs zum Test: eine **[Verbatim Vi560](https://ebay.us/JKLBbx)** und eine **[Samsung EVO SSD 860](https://ebay.us/AjyRqF)**. Beide nutzen die SATA-Schnittstelle, unterscheiden sich aber im Controller und Cache-Verhalten unter anhaltender I/O-Last.

{{< admonition type="note" title="Zwei M.2 SATA-Disks im Vergleich" open=true >}}
Wichtig zu erwähnen: **Beide Testkandidaten sind M.2-SATA-SSDs.** Obwohl sie denselben Formfaktor und dieselbe Bus-Schnittstelle nutzen, zeigen sich im Detai
l spannende Unterschiede:
* **[Verbatim Vi560 (M.2 SATA)](https://ebay.us/JKLBbx):** Ein preiswerter Einsteiger-Allrounder. Bei kurzen Zugriffen solide, gerät das schlankere Cache-Management bei anhaltend hohen Schreib- und Lesezyklen (wie beim Kompilieren) jedoch schneller an Leistungsgrenzen.
* **[Samsung 860 EVO SSD (M.2 SATA)](https://ebay.us/AjyRqF):** Ein bewährter Klassiker im M.2-Gewand. Dank eines ausgereiften Controllers und eines besseren DRAM-Caches hält sie die Transferraten auch unter dauerhafter I/O-Last extrem stabil.
{{< /admonition >}}

RAM und SSDs gibts preiswert bei eBay, siehe entsprechende Links oben.

{{< admonition type="warning" title="M.2 SATA vs. M.2 NVMe PCIe: Achte unbedingt auf den richtigen Slot!" open=true >}}
Bevor du eine neue M.2-SSD kaufst, solltest du genau prüfen, welchen Standard der M.2-Steckplatz deines Mainboards oder Laptops unterstützt. Obwohl beide Formfaktoren identisch aussehen können, nutzen sie völlig unterschiedliche Technologien. **Wichtiger Hinweis zum Gerät:** Der hier getestete Geekom IX12 besitzt einen M.2 SATA 2280 Slot und unterstützt keine NVMe PCIe SSDs!

* **M.2 SATA (AHCI):**
  * **Geschwindigkeit:** Maximal ~550 MB/s (durch das SATA-III-Protokoll begrenzt).
  * **Einkerbung (Keying):** Nutzt meist den **B+M-Key** (zwei Einkerbungen).
  * **Einsatzbereich:** Ältere Systeme, spezialisierte Mini-PCs (wie der Geekom IX12) oder günstige Einstiegs-SSDs.

* **M.2 NVMe (PCIe):**
  * **Geschwindigkeit:** Deutlich schneller (3.500 bis über 7.000 MB/s via PCIe 3.0/4.0/5.0).
  * **Einkerbung (Keying):** Nutzt meist den **M-Key** (eine Einkerbung).
  * **Einsatzbereich:** Moderne PCs, Laptops, Gaming-Rigs und Workstations.

---

### Worauf du beim Kauf achten musst:

1. **Kompatibilität prüfen:** Ein M.2-Slot ist **nicht** automatisch abwärts- oder aufwärtskompatibel. Ein rein PCIe/NVMe-fähiger M.2-Slot erkennt eine M.2 SATA-SSD oft gar nicht (und umgekehrt).
2. **Handbuch konsultieren:** Schau im Handbuch deines Mainboards oder Laptops nach den genauen Spezifikationen des Slots (z. B. *"M.2 Slot 1 supports PCIe 4.0 x4 & SATA mode"*) – manche Slots unterstützen beide Standards, andere nur einen von beiden.
3. **Keying beachten:** 
   * **M-Key:** Für NVMe/PCIe (kann physisch in M-Slots gesteckt werden).
   * **B+M-Key:** Meist für SATA (passt physisch in B- und M-Slots, benötigt aber Protokoll-Unterstützung im BIOS/Chipsatz).
4. **Länge prüfen:** Achte auf die unterstützte Formfaktor-Länge (z. B. 2280, 2242, 2230). 2280 ist der Standard für Desktop-PCs und viele Mini-PCs.
{{< /admonition >}}


Aber erstmal ging es darum, herauszufinden, ob die wesentlichen Komponenten wichtig für den Betrieb als Desktop funtionieren:

 * Grafik (X11) (inteldrm)
 * Netzwerkkarte (Wi-Fi war mir nicht so wichtig, da ich CAT 7 am Schreibtisch anliegen habe) (4x igc)
 * Audio (azalia)
 * USB Ports

Und das tat es alles sehr wohl. Die "not configured" Komponenten sind für den Betrieb als Desktop nicht wichtig. WiFi kann einfach ausgetauscht werden. Das komplette `dmesg` im Kasten.


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
acpicpu2 at acpi0: C3(200@1048 mwait.1@0x60), C2(350@127 mwait.1@0x21), C1(1000@1 mwait.1), PSS
acpicpu3 at acpi0: C3(200@1048 mwait.1@0x60), C2(350@127 mwait.1@0x21), C1(1000@1 mwait.1), PSS
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

Unter der Haube des Geekom IX12 werkelt ein Intel N95 Prozessor, der auf Intels energieeffizienter [Alder Lake N](https://en.wikipedia.org/wiki/Alder_Lake) Architektur basiert. Dieser System-on-a-Chip setzt ausschließlich auf vier effiziente Gracemont-Kerne ("E-Cores"), kommt ohne stromhungrige Performance-Kerne aus und eignet sich mit einer maximalen Thermal Design Power (TDP) von wenigen Watt perfekt für lüfterlose Mini-PCs.

---

## Das Frequenz-Rätsel & die Lösung der Mailingliste

Beim ersten Antesten von `sysctl hw.cpuspeed` schreckte mich ein Wert ab: Er verharrte starr bei **1701 MHz**. Weder `sysctl hw.perfpolicy=high` noch synthetische Benchmarks veränderten diese Zahl. Auch der Gang ins BIOS – zum Deaktivieren von *Intel Speed Shift* oder zur Freischaltung des Turbo Mode – schien auf den ersten Blick nichts an der Anzeige zu ändern.

Eine Nachfrage auf der OpenBSD-Mailingliste brachte schlussendlich das entscheidende Licht ins Dunkel: `hw.cpuspeed` liest lediglich die primären ACPI-Tabellen aus. Die 1701 verkörpert den Turbo Mode. Die echten, aktuellen Live-Frequenzen der Kerne lassen sich jedoch exakt über **`sysctl hw.sensors`** beobachten!

{{< admonition type="tip" title="Erkenntnis der Community" open=true >}}
OpenBSD pusht die Turbo-Frequenz bei Lastspitzen dynamisch bis zum Maximum von hier 3.2GHz. Unter starker Multi-Core-Dauerlast greifen die BIOS Power Limits (PL1/PL2) zum Schutz der Passivkühlung und pendeln das System stabil auf 2.7GHz ein.
{{< /admonition >}}

Die Messreihen mit `sysctl hw.sensors` bestätigen das perfekte Verhalten:

* **Leerlauf:** Traumhafte 38 °C bei minimalen **800 MHz** auf allen Kernen.
* **Kurzer Burst (4x sha256 -tt):** Die CPU schießt sofort kurz auf **3,35 GHz** hoch (bei ca. 54 °C).
* **Dauerlast (4x sha256 -tt):** Nach Greifen des Sustained Power Limits (PL1) pendeln sich alle 4 Kerne stabil bei **2,70 GHz** und kühlen 49 °C ein.
* **Teillast (2x sha256 -tt):** Da das 15W-Budget nur auf zwei Kerne verteilt wird, bleiben die Frequenzen dauerhaft hoch zwischen **3,00 GHz und 3,25 GHz** (bei ca. 62 °C).

Das Frequenzmanagement arbeitet also absolut "as designed" und holt aus den vier Efficient-Cores des Intel N95 das Optimum heraus.

---

## Performance-Test: Kompilieren unter OpenBSD

Keine Rätsel mehr – wie schlagen sich die beiden Rechner nun im direkten Vergleich? 

CPU-intensive Aktivitäten gewinnt der alte HP: 4x 3.2 GHz Dauerleistung unter Volllast sind im reinen Rechenpass schlicht mehr als 4x 2.7 GHz All-Core-Takt. Bei gemischten Workloads (CPU, Memory und I/O) schneidet der Geekom hingegen extrem passabel ab und ist bei hohem I/O-Anteil sogar spürbar schneller!

Um faire Bedingungen zu schaffen, liefen beide Maschinen mit identischen Vorgaben (32 GB RAM, `hw.smt=1`, `apm -H`, `noatime` Mount-Optionen).

### 1. OpenJFX (inkl. WebKit)

Das Kompilieren von OpenJFX stellt eine gigantische CPU-Dauerbelastung dar.

| System / Setup | `make` (Real Time) | `make fake` (Real Time) |
| :--- | :--- | :--- |
| **HP Compaq Pro** (SSD) | **172m 11.28s** | 0m 02.20s |
| **Geekom IX12** (Samsung SSD) | **178m 15.70s** | 0m 01.19s |
| **Geekom IX12** (Verbatim SSD) | **182m 29.37s** | 0m 01.32s |

Hier liegt der alte HP knapp 6 Minuten vorne, da er seine 3,2 GHz ungebremst durchkühlen kann. Der Geekom hält mit seinen 2,7 GHz All-Core-Takt aber bravourös mit. Aber was sind bei knapp 3 Stunden schon 6 Minuten unterschied.

### 2. Autopsy (`make clean && make fake`)

Ein gemischter Build-Prozess mit viel Verzeichnisstruktur-, Skript- und Datenträger-I/O:

| System / Setup | Real Time | User Time | System Time |
| :--- | :--- | :--- | :--- |
| **HP Compaq Pro** | 8m 35.53s | 6m 45.57s | 2m 10.93s |
| **Geekom IX12** (Samsung SSD) | **6m 08.76s** | 6m 44.93s | 1m 19.31s |
| **Geekom IX12** (Verbatim SSD) | **6m 37.05s** | 6m 34.27s | 1m 21.61s |

Hier gewinnt der [Geekom IX12](https://www.awin1.com/cread.php?awinmid=32589&awinaffid=3025379&ued=https%3A%2F%2Fwww.geekom.de%2Fgeekom-ix12-luefterloser-mini-pc%2F) haushoch: Er nimmt dem alten HP **fast 2,5 Minuten** ab. Die modernere Plattform, schnellerer Arbeitsspeicher, die neuere Bus-Architektur und die Samsung SSD (welche der Verbatim nochmals knapp 30 Sekunden abnimmt) sorgen für einen ordentlichen I/O-Vorteil.

---

## Fazit & Ausblick

Der Umstieg vom alten HP Compaq Pro auf den [Geekom IX12](https://www.awin1.com/cread.php?awinmid=32589&awinaffid=3025379&ued=https%3A%2F%2Fwww.geekom.de%2Fgeekom-ix12-luefterloser-mini-pc%2F) war ein voller Erfolg! Als DVD Laufwerksersatz, für den Fall der Fälle, gabs letztens noch ein mobiles DVD Laufwerk als 3EUR Schnäppchen vom Flohmarkt ;)

* **Lautlos:** Absolute Stille am Schreibtisch – kein andauerndes Lüftergeräusch mehr!
* **Effizient:** Der Stromverbrauch ist im Vergleich zur alten Ivy-Bridge-Plattform verschwindend gering.
* **Flott im Alltag:** Bei alltäglichen Aufgaben, I/O-lastigen Tasks und kurzen Bursts reagiert der Geekom durch den höheren Single-Core-Turbo sogar spürbar zackiger.

Kleines Manko: Ein paar mehr USB Ports wären schön gewesen, so muss der Monitor erstmal als USB Hub herhalten.

Obwohl der Geekom primär als Industrie-PC (optimal als Firewall oder Router) beworben wird: Mit etwas mehr RAM und einer größeren SSD macht das Gerät auch als Desktop eine hervorragende Figur! Für die Zukunft bleibt das Setup eine ideale Basis – ob weiterhin als geräuschlose Daily-Driver-Workstation oder später als leistungsfähiger 4x 2.5G OpenBSD-Router im Home-Lab. Die Hardware hat sich im Testparcours definitiv bewährt!

