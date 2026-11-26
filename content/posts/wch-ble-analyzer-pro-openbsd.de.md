---
license: "MIT"
title: "WCH BLE Analyzer Pro unter OpenBSD: Drei Kanäle gleichzeitig mit Kismet und wch_capture"
subtitle: "BLE-Advertising parallel auf Kanal 37, 38 und 39 mitschneiden – an nur einem USB-Port"
description: "Erfahrungsbericht zum WCH BLE Analyzer Pro unter OpenBSD: BTLE-Advertising-Kanäle, Kismet-Datasource, wch_capture"
date: 2026-11-26T22:02:18+01:00
lastmod: 2026-11-26T22:02:18+01:00
draft: false
images:
  - "/images/WCH_BLE_OpenBSD.jpg"
  - "/images/wch-ble-analyzer-pro-front.jpg"
  - "/images/wch-ble-analyzer-pro-back.jpg"
  - "/images/Kismet_WCH_BLE_Data_Sources.png"
  - "/images/Kismet_WCH_BLE_in_Action.png"
  - "/images/WCH_BLE_Analyzer_Wireshark.png"

tags:
  - "OpenBSD"
  - "BTLE"
  - "BLE"
  - "Kismet"
  - "WCH"
  - "Sniffing"
  - "Ports"
categories:
  - "OpenBSD"
  - "Hardware"

featuredImage: "/images/WCH_BLE_OpenBSD.jpg"
featuredImagePreview: "/images/WCH_BLE_OpenBSD.jpg"

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
    - "/images/WCH_BLE_OpenBSD.jpg"
    - "/images/wch-ble-analyzer-pro-front.jpg"
    - "/images/wch-ble-analyzer-pro-back.jpg"
    - "/images/Kismet_WCH_BLE_Data_Sources.png"
    - "/images/Kismet_WCH_BLE_in_Action.png"
    - "/images/WCH_BLE_Analyzer_Wireshark.png"
---

## Ja wieso das denn? ;)

Als ich kürzlich **[Kismet](https://kismetwireless.net/)** auf das [neueste Release geupdated habe](/openbsd-ports-updating-spree-nov26/), ist mir die neue Data Source [https://www.kismetwireless.net/docs/readme/datasources/bluetooth-wch-ble-analyzer-pro/](https://www.kismetwireless.net/docs/readme/datasources/bluetooth-wch-ble-analyzer-pro/) aufgefallen. Da sich die Data Source ohne Weiteres bauen ließ, war ich neugierig genug und musste mir so ein Teil bestellen – erhältlich unter anderem bei [https://www.olimex.com/Products/RISC-V/WCH/WCH-BLE-Analyzer-PRO/](https://www.olimex.com/Products/RISC-V/WCH/WCH-BLE-Analyzer-PRO/).

## Warum drei Kanäle gleichzeitig den Unterschied machen

Bei Bluetooth Low Energy (BLE) spielen im 2,4-GHz-Band insgesamt 40 Kanäle (0 bis 39) eine Rolle, doch für die Geräteerkennung und den Verbindungsaufbau sind vor allem die **drei Advertising-Kanäle** entscheidend: **Kanal 37 (2402 MHz)**, **Kanal 38 (2426 MHz)** und **Kanal 39 (2480 MHz)**. Genau hier senden Geräte ihre Advertisements aus, beantworten Scan-Requests oder stoßen Verbindungen an.

Ein klassischer Single-Chip-BLE-Sniffer muss zwischen diesen Kanälen hin- und herspringen. Das reicht für einen schnellen Überblick, führt aber leicht dazu, dass kurzlebige Pakete oder exakte Timing-Abläufe auf einem gerade nicht abgehörten Kanal durchrutschen. Wer alle drei Advertising-Kanäle lückenlos im Blick behalten möchte, bräuchte dafür mit herkömmlichen Sniffern **drei separate Sniffer an drei USB-Ports** – inklusive Kabelgewirr und knapp werdenden Ports am Rechner.

Der **WCH BLE Analyzer Pro** löst dieses Problem elegant: Im Inneren arbeiten **drei CH582F RISC-V-Chips** zusammen mit einem **CH334 USB-Hub**. Jeder Chip kann einen Kanal bedienen, sodass die drei Advertising-Kanäle parallel belauscht werden können – und das Ganze benötigt **nur einen einzigen USB-Port** am Rechner, wobei hinten am Hub sogar noch ein weiteres Gerät angesteckt werden kann.

{{< figure src="/images/wch-ble-analyzer-pro-front.jpg" alt="WCH BLE Analyzer Pro Frontseite mit USB Anschluss" caption="WCH BLE Analyzer Pro Frontseite mit USB Anschluss" class="ma0 w-75" >}}
{{< figure src="/images/wch-ble-analyzer-pro-back.jpg" alt="WCH BLE Analyzer Pro Rückseite mit Antennen" caption="WCH BLE Analyzer Pro Rückseite mit Antennen" class="ma0 w-75" >}}


{{< admonition type="info" title="Nur ein USB-Port am Rechner – plus Anschluss hinten am Hub" open=true >}}
Statt drei getrennter Sniffer an drei USB-Ports belegt der WCH BLE Analyzer Pro nur einen einzigen Host-Port und bietet hinten am integrierten USB-Hub sogar noch Platz für ein weiteres Gerät. Gerade am Notebook oder einem kompakten Testrechner ist das angenehm aufgeräumt.
{{< /admonition >}}

---

## Kismet, wch_capture und OpenBSD

### Mit Kismet direkt starten

Wenn die Data Source sauber gebaut ist, lässt sich mit Kismet zügig ein Mitschnitt in `pcapng` anlegen. Für erste Experimente mit Debug-Ausgabe bietet sich ein eigenes Verzeichnis an:

{{< highlight shell >}}
mkdir -p ble-logs && kismet -T pcapng -p ble-logs -t ble-test --debug
{{< /highlight >}}
Damit lassen sich BLE-Pakete direkt über Kismet im pcapng Format abspeichern, und im Anschluss weiter untersuchen.

Dank libusb werden die Data Sources für die 3 Kanäle automatisch Erkannt. Nur noch in Kismet aktivieren, und los gehts.

{{< figure src="/images/Kismet_WCH_BLE_Data_Sources.png" alt="Kismet wch-btle-mcu Data Sources" caption="Kismet wch-btle-mcu Data Sources" class="ma0 w-75" >}}
{{< figure src="/images/Kismet_WCH_BLE_in_Action.png" alt="WCH BLE Analyzer Pro BTLE Überwachung in Aktion" caption="WCH BLE Analyzer Pro BTLE Überwachung in Aktion" class="ma0 w-75" >}}


### `wch_capture` und saubere `.pcap`-Dateien

Neben Kismet bin ich auf das Repository [https://github.com/xecaz/BLE-Analyzer-pro-linux-capture](https://github.com/xecaz/BLE-Analyzer-pro-linux-capture) gestoßen. Das Werkzeug bringt eine übersichtliche CLI mit, über die sich unter anderem PHY, Filter, Kanalwahl, Custom-2.4G-Parameter sowie die Ausgabe auf `stdout` oder in eine `.pcap`-Datei steuern lassen:

{{< highlight text >}}
wch_capture                                                                                                                                                                          
Nothing to do – use -v and/or -w FILE.pcap
WCH BLE Analyzer PRO Linux Capture tool by Xecaz 2026!

Usage: wch_capture [OPTIONS]

Options:
  -v            Print packets to stdout
  -w FILE.pcap  Write PCAP (DLT 256, BLE LL + phdr)
  -p PHY        PHY: 1=1M (default), 2=2M, 3=CodedS8, 4=CodedS2
  -i ADDR       Initiator MAC filter  (AA:BB:CC:DD:EE:FF)
  -a ADDR       Advertiser MAC filter (AA:BB:CC:DD:EE:FF)
  -k KEY        LTK, 32 hex chars
  -K PASSKEY    BLE passkey (6-digit decimal)
  -2            Custom 2.4G mode (default: BLE monitor)
  -c CHAN       Channel 0-39: BLE adv 37/38/39 or 0=all (auto per MCU); 2.4G raw
  -A AADDR      2.4G access addr (hex, e.g. 8E89BED6)
  -C CRCINIT    2.4G CRC init (6 hex chars, e.g. 555555)
  -W WHITEN     2.4G whitening init (hex byte)
  -h            Show this help

Capture stops on SIGINT (Ctrl+C) or SIGTERM.
{{< /highlight >}}

Unter OpenBSD lässt sich `wch_capture` einfach bauen. Es benötigt lediglich `gmake` und `libusb-1.0` Pakete installiert. Bauen einfach mit `gmake CFLAGS=-I/usr/local/include`.

Dann lässt es sich auch schon direkt ausführen, etwa mit Live-Ausgabe im Terminal und Schreiben einer `.pcap`-Datei:

{{< highlight shell >}}
sudo /usr/local/bin/wch_capture -w output.pcap -v
{{< /highlight >}}

Das schreibt auch schöne `.pcap`-Dateien, mit denen sich weiterarbeiten lässt.

{{< admonition type="tip" title="PCAP-Dateien von Kismet und wch_capture in Kismet öffnen" open=true >}}
Sowohl Captures aus Kismet als auch `.pcap`-Dateien von `wch_capture` lassen sich einfach in Wireshark, ohne weitere Plugins zu benötigen, öffnen. So lassen sich Rohmitschnitte aus `wch_capture` bequem im gewohnten Wireshark UI analysieren.
{{< /admonition >}}

{{< figure src="/images/WCH_BLE_Analyzer_Wireshark.png" alt="BTLE Traffic Mitschnitt in Wireshark" caption="BTLE Traffic Mitschnitt in Wireshark" class="ma0 w-75" >}}

### Port-Review auf der OpenBSD `ports@`-Mailingliste

Damit `wch_capture` unter OpenBSD sauber gepflegt und reproduzierbar baubar bleibt, ist ein entsprechender Port zum Review an die OpenBSD `ports@`-Mailingliste gesendet. Gerade bei spezialisierten Funk- und Capture-Werkzeugen ist ein sauberer Port unter OpenBSD Gold wert.

---

## Fazit

Was soll man sagen, unterm Strich ein kleines robustes Gerät, nicht übermäßig teuer – und mit Kismet und `wch_capture` fast schon zu einfach zu benutzen ;)

