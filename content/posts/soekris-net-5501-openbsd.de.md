---
title: "Soekris net5501 und OpenBSD"
subtitle: "Der legendäre Embedded-Klassiker im Langzeittest"
date: 2026-09-16T21:30:00+02:00
lastmod: 2026-09-16T21:30:00+02:00
draft: false
description: "Ein nostalgischer und sachlicher Rückblick auf die Soekris net5501 als extrem sparsame OpenBSD-Plattform für Netzwerk- und Konsolendienste."
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

Wer sich schon länger in der Welt der Unix-Sicherheits-Firewalls und minimalistischen Server-Systeme bewegt, erinnert sich mit einer gewissen Nostalgie an die graugrünen oder dunklen Gehäuse von Soekris Engineering. Jahrelang war die **[Soekris net5501](https://ebay.us/IQQupe#affiliate)** der Goldstandard für hochverlässliche, lüfterlose Embedded-Router und Netzwerk-Appliances unter OpenBSD.

Nachdem ich vor einiger Zeit über den [Fujitsu Futro S920 und OpenBSD](/fujitso-futro-s920-openbsd/) geschrieben habe, werfen wir heute einen Blick auf ein echtes Urgestein der x86-32bit-Ära. Die Firma Soekris hat ihre Pforten zwar leider vor Jahren geschlossen, aber die Hardware läuft und läuft – auch wenn sie heute eher Liebhaberstatus besitzt.

<figure class="ma0 w-75">
  <img src="/images/Soekris-Back.jpg" alt="Soekris net5501 Rückansicht">
  <figcaption>Die Soekris net5501: Rückansicht</figcaption>
</figure>

---

## Hardware-Eigenschaften und Schnittstellen

Die net5501 setzt auf den AMD Geode LX SOC – eine Plattform, die extrem energieeffizient arbeitet, aber leistungstechnisch natürlich Welten von moderner Hardware entfernt ist. Dafür besticht das Board durch seine Erweiterbarkeit und fantastische Zuverlässigkeit im Dauerbetrieb.

{{< admonition type="info" title="Hardware-Übersicht & Spezifikationen" >}}
* **Prozessor:** AMD Geode LX 800 (500 MHz x86, i386-Architektur)
* **Arbeitsspeicher (RAM):** 512 MB DDR-SDRAM (fest verlötet)
* **Netzwerk:** 4x Fast Ethernet Ports (`vr0` bis `vr3`, VIA VT6105M Rhine III)
* **Speicher:** CompactFlash-Steckplatz (intern, dient als Boot-Medium/Festplatte) sowie ein primärer IDE-Kanal
* **Serielle Ports:** Standard-Konsolenport via RS-232 (`com0`), erweiterbar über interne Head-Module oder PCI-Karten
* **Erweiterbarkeit:** 1x 32-Bit PCI-Steckplatz und MiniPCI-Slot (z. B. für Kryptokarten, WLAN oder Multiplex-Seriell-Karten)
* **Krypto-Beschleunigung:** Onboard AMD Geode LX Crypto Engine (`glxsb0`) für Hardware-RNG und AES-Verschlüsselung
* **Stromverbrauch:** Phänomenale **4 bis 6 Watt** Leistungsaufnahme über ein simples externes Netzteilelement (bzw. wie hier in der Rackmount Variante, eingebaut)
{{< /admonition >}}


<figure class="ma0 w-75">
  <img src="/images/Soekris-Innen.jpg" alt="Soekris net5501 Innenansicht">
  <figcaption>Das Innenleben der Soekris net5501 inklusive CompactFlash-Karte</figcaption>
</figure>

---

## Hardware-Erkennung unter OpenBSD (i386)

Obwohl die 32-Bit-i386-Architektur heutzutage von vielen Linux-Distributionen schrittweise aufgegeben wird, hält das OpenBSD-Projekt dem alten Silizium nach wie vor die Treue. Die net5501 läuft mit dem Standard-i386-Kernel absolut einwandfrei.

Hier ist der Auszug aus dem Boot-Log (`dmesg`) unter OpenBSD 8.0-beta:

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

{{< admonition type="note" title="Serielle Konsole als Standard" >}}
Die Soekris-Hardware besitzt keinen Grafikausgang (VGA/HDMI). Die Steuerung erfolgt rein seriell über `com0` (9600 Baud Standard im comBIOS von Soekris. Kann man dort natürlich ändern, auf bis zu 115200 Baud). In OpenBSD muss das dann entsprechend in `/etc/boot.conf`  und `/etc/ttys` konfiguriert werden.
{{< /admonition >}}

{{< admonition type="warning" title="Wichtiger Hinweis: Schreibzugriffe auf CompactFlash reduzieren" >}}
Da das System im Gezeigten auf einer 2 GB CompactFlash-Karte (`wd0`: SanDisk SDCFH-002G) läuft, sollte man EXTREM vorsichtig mit häufigen Schreibzugriffen sein, um das Flash-Medium nicht zügig zu verschleißen.

**Lösung:** Verzeichnisse wie `/var/log`, `/tmp` oder `/var/run` über den Arbeitsspeicher via MFS (Memory File System) mounten oder Log-Einträge via `syslogd` direkt an einen entfernten Log-Server schicken. Alternativ kann man aber auch eine SATA Disk, oder SSD einbauen.
{{< /admonition >}}

---

## Einsatzzwecke im Rückblick

Aufgrund der überschaubaren 500 MHz Taktfrequenz und den 100 MBit/s Netzwerkschnittstellen (`vr0`-`vr3`) war die Soekris bei mir primär für spezialisierte Verwaltungsaufgaben zuständig:

1. **Serieller Terminalserver:** Durch die eingebaute Sunix-PCI-Karte standen ganze 8 zusätzliche RS-232-Schnittstellen (`com4` bis `com11`) bereit. Perfekt, um die Konsolenports von Switchen, Routern oder anderen Headless-Servern zentral via `cu` abzugreifen.
2. **Out-of-Band-Management:** Als kleiner, robuster Notfall-Zugangsknoten.
3. **Netzwerkserver:** Als DNS, DHCP, TFTP etc. Server.

Ein besonderer Clou meines Setups war der Einsatz einer [Sunix Multi-Port Serial PCI-Erweiterung](https://ebay.us/I3tMXl#affiliate) (`puc0`), wodurch die Kiste zum ultimativen Multi-Port-Konsolenserver wurde.

<figure class="ma0 w-75">
  <img src="/images/Sunix-8-Port-Serial.jpg" alt="Sunix 8 Port Serial Card PCI">
  <figcaption>Sunix 8 Port Serial Card PCI</figcaption>
</figure>

---

## Fazit & Ausblick

Ist eine [Soekris net5501](https://ebay.us/IQQupe#affiliate) im aktuellen Zeitalter noch praxisnah? Für moderne Gigabit-Leitungen mit tiefgehender Paketinspektion (PF) ist der Geode LX 800 schlicht zu langsam. Auch das Fehlen von 64-Bit-Unterstützung und Gigabit-Schnittstellen zeigt das Alter der Plattform.

Wer jedoch ein stoisches, absolut lautloses Stück Hardwaregeschichte für minimalistische Aufgaben oder als serieller Server sucht, findet gebrauchte Geräte gelegentlich auf Marktplätzen wie [eBay](https://ebay.us/EqGEBs#affiliate). Es beeindruckt immer wieder, mit wie wenig Rechenleistung und Speicherplatz ein vollwertiges, hochsicheres OpenBSD-System auskommen kann!

