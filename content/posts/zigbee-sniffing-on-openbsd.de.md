---
title: "Zigbee-Sniffing unter OpenBSD: Deep Dive mit dem TI CC2531"
subtitle: "Ein Update zum Zigbee- & BTLE-Sniffing: Von Upstream-Patches für Kismet bis hin zu whsniff auf OpenBSD."
date: 2026-05-08T21:30:33+02:00
lastmod: 2026-05-08T21:30:33+02:00
draft: false
author: ""
authorLink: ""
description: "Wie ich Kismet-Mutex-Bugs behoben, whsniff auf OpenBSD portiert und endlich IEEE 802.15.4-Traffic lauschen konnte."
license: "MIT"
images: [ "/images/sniffing_802154_openbsd.png", "/images/ticc2531.jpg", "/images/kismet_zigbee.png", "/images/wireshark_802154.png" ]

tags: [ "OpenBSD", "Zigbee", "Sniffing", "Kismet", "Wireshark", "CC2531", "802.15.4"]
categories: [ "Tech" ]

featuredImage: "/images/sniffing_802154_openbsd.png"
featuredImagePreview: "/images/sniffing_802154_openbsd.png"

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
---

In meinem [vorherigen Post](https://buzzdeee.reitenba.ch/btle-zigbee-sniffing/) habe ich den [nRF52840 nice!Nano](https://www.ebay.de/itm/157602300188?amdata=enc%3AAQALAAAAoGfYFPkwiKCW4ZNSs2u11xC1d7FAa6flkF10AXmFbxjEfI9HUlmk3Monv69OxPg8gzKLTecE0Z2vlr4hjmU3PSz%2BOj0UTEToVcSBX7mUGUDjoywT1OguTvGKMjWykpZ6tqEnlcktfEpWSKxwXuXD1%2FTG%2BTU9F%2BrtgwVj7Ssv0ufYQ%2Br973tF29nUr5fy1WteZZFk%2FZrEkUWv6cM8oloBCF4%3D&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF52840&toolid=10001&mkevt=1) als Kombi-Lösung für BTLE- und Zigbee-Sniffing unter die Lupe genommen. Während BTLE tadellos lief, herrschte bei Zigbee in Kismet und Wireshark Funkstille.

Um Hardware-Limits als Fehlerquelle auszuschließen, bin ich auf einen echten Klassiker umgestiegen: den **[Texas Instruments CC2531 USB Dongle](https://www.ebay.de/itm/404481824304?var=674447321503&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=TICC2531&toolid=10001&mkevt=1)**.

## Die Hardware: TI CC2531

Ich habe mir von [eBay](https://www.ebay.de/itm/404481824304?var=674447321503&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=TICC2531&toolid=10001&mkevt=1) einen [TI CC2531](https://www.ebay.de/itm/404481824304?var=674447321503&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=TICC2531&toolid=10001&mkevt=1) besorgt, der bereits mit der Sniffer-Firmware und einer ordentlichen externen Antenne ausgestattet war. Im Vergleich zum [nice!Nano](https://www.ebay.de/itm/157602300188?amdata=enc%3AAQALAAAAoGfYFPkwiKCW4ZNSs2u11xC1d7FAa6flkF10AXmFbxjEfI9HUlmk3Monv69OxPg8gzKLTecE0Z2vlr4hjmU3PSz%2BOj0UTEToVcSBX7mUGUDjoywT1OguTvGKMjWykpZ6tqEnlcktfEpWSKxwXuXD1%2FTG%2BTU9F%2BrtgwVj7Ssv0ufYQ%2Br973tF29nUr5fy1WteZZFk%2FZrEkUWv6cM8oloBCF4%3D&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF52840&toolid=10001&mkevt=1) ist der CC2531 ein dediziertes 802.15.4-Radio – also die perfekte Referenz, um Zigbee-Traffic auf den Grund zu gehen.

{{< figure
    src="/images/ticc2531.jpg"
    alt="Texas Instruments CC2531 mit Antenne"
    caption="Der TI CC2531 – klein, aber oho dank externer Antenne."
    class="ma0 w-75"
>}}

## Kismet gezähmt: Mutex-Chaos und Libusb-Fixes

Damit die `ti_cc_2531`-Quelle in Kismet unter OpenBSD stabil läuft, musste ich tiefer in den Code eintauchen. Das System quittierte den Dienst regelmäßig mit `ABORT`-Meldungen, da Mutexes unsauber gehandhabt wurden (Double-Unlocks oder Versuche, nie gesperrte Mutexes freizugeben).

Zudem gab es Reibungspunkte mit `libusb`: Da OpenBSD `libusb_detach_kernel_driver()` nicht unterstützt, musste der Code lernen, diesen Fehler schlicht zu ignorieren. Mit ein paar zusätzlichen `libusb_ref_device`-Aufrufen war dann auch das Referenz-Management der Hardware sauber.

**Gute Nachrichten: Die Patches wurden heute offiziell übernommen!** Wer sich die Details anschauen möchte, findet hier den [PR](https://github.com/kismetwireless/kismet/pull/604).

Dank der Fixes erkennt Kismet den Dongle nun einwandfrei und spürt Zigbee-Geräte in der Umgebung auf.

{{< figure
    src="/images/kismet_zigbee.png"
    alt="Kismet erkennt Zigbee-Geräte unter OpenBSD"
    caption="Endlich Traffic: Kismet bei der Arbeit unter OpenBSD."
    class="ma0 w-75"
>}}

## Die Brücke zu Wireshark: whsniff

Discovery ist schön, aber für die echte Analyse führt kein Weg an Wireshark vorbei. Um die Daten vom CC2531 dorthin zu tunneln, kam [whsniff](https://github.com/homewsn/whsniff) zum Einsatz.

Eigentlich für Linux geschrieben, läuft es nach einem kleinen Patch (siehe [Pull Request #25](https://github.com/homewsn/whsniff/pull/25)) nun auch unter OpenBSD. Damit lässt sich der Traffic direkt in Wireshark einspeisen:

{{< highlight bash "linenos=table" >}}
# whsniff via doas direkt in Wireshark pipen
doas whsniff -c 11 | wireshark -k -i -
{{< /highlight >}}

Endlich kann ich 802.15.4-Datenverkehr live und in Farbe analysieren.

{{< figure
    src="/images/wireshark_802154.png"
    alt="Wireshark sniffing IEEE 802.15.4"
    caption="Live-Analyse der Pakete in Wireshark."
    class="ma0 w-75"
>}}

{{< admonition tip "Tipps für Zigbee in Wireshark" >}}
Den Traffic zu sehen ist der erste Schritt, ihn zu verstehen der zweite:

1. **Verschlüsselung**: Da Zigbee in der Regel verschlüsselt ist, hilft manchmal der Standard-Link-Key **ZigBeeAlliance09**. Diesen könnt ihr unter **Edit -> Preferences -> Protocols -> ZigBee -> Pre-configured Keys** eintragen.
   * **Key (Hex):** `5A:69:67:42:65:65:41:6C:6C:69:61:6E:63:65:30:39`
   * Falls das nicht reicht, braucht ihr den spezifischen Netzwerkschlüssel eures Controllers (z. B. aus Zigbee2MQTT).
2. **Rauschen ausblenden**: Im 2.4-GHz-Band ist viel los. Pakete mit kaputter Checksumme (FCS) werden oft verworfen. Um sie dennoch in Wireshark zu dekodieren, deaktiviert: **Edit -> Preferences -> Protocols -> IEEE 802.15.4 -> "Dissect only good FCS"**.
{{< /admonition >}}

## Das nice!Nano-Rätsel

Und was ist mit dem [nice!Nano/nRF52840](https://www.ebay.de/itm/157602300188?amdata=enc%3AAQALAAAAoGfYFPkwiKCW4ZNSs2u11xC1d7FAa6flkF10AXmFbxjEfI9HUlmk3Monv69OxPg8gzKLTecE0Z2vlr4hjmU3PSz%2BOj0UTEToVcSBX7mUGUDjoywT1OguTvGKMjWykpZ6tqEnlcktfEpWSKxwXuXD1%2FTG%2BTU9F%2BrtgwVj7Ssv0ufYQ%2Br973tF29nUr5fy1WteZZFk%2FZrEkUWv6cM8oloBCF4%3D&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF52840&toolid=10001&mkevt=1)? Hier habe ich unter OpenBSD leider immer noch keine stabilen 802.15.4-Frames einfangen können. Vielleicht liegt es an der lausigne Antenne oder der Firmware.

Um das zu klären, habe ich mir einen **[Sonoff Dongle M](https://www.ebay.de/itm/388971238285?amdata=enc%3AAQALAAAAoGfYFPkwiKCW4ZNSs2u11xBWFdbfFYp4er2kjyno9qmO593oK669JSSjjTgWvi6tTbapsCl%2FVp2mI58X9g82SgP81da1K3tWO4DD1pBdPF3sW8qHl6SAyOQrvVfMDGTMK23eQrjJoSroW1eTkSWk%2B2gMNy9psH35o8kjxRGrjbpxdfL5BH8uSE4Y44WKK5Swaf31ULdlZn1RqlWAhOr4IgM%3D&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Sonoff&toolid=10001&mkevt=1)** bestellt. Sobald mein eigenes Home-Automation-Lab steht, kann ich ein Zigbee-Gerät direkt neben den nice!Nano legen und schauen, ob es ein reines Reichweiten-Thema ist.

## Fazit

Der CC2531 ist nach wie vor ein klasse Werkzeug für die OpenBSD-Toolbox, besonders da der Kismet-Support jetzt stabil läuft. Auch wenn die Hardware betagt ist: Die externe Antenne macht ihn zum perfekten Referenzgerät auf jedem Schreibtisch.

Ich habe heute außerdem ein Kismet-Port-Update sowie einen neuen Port für whsniff zur Begutachtung an ports@ geschickt und hoffe, dass diese bald in den Ports-Tree einziehen können.

Sobald der Sonoff-Dongle da ist, gibt es das nächste Update!
