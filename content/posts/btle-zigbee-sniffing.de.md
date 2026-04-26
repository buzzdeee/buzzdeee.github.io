---
title: "Unsichtbare Signale abfangen: BTLE und Zigbee auf OpenBSD"
subtitle: "Professionelle Funk-Analyse mit nRF51822 und nRF52840"
description: "Erkunde BTLE- und Zigbee-Traffic auf OpenBSD mit nRF52840-Hardware, Kismet und Wireshark für drahtlose Echtzeitanalyse."
date: 2026-04-24T16:24:29+02:00
lastmod: 2026-04-24T16:24:29+02:00
draft: false
tags: ["OpenBSD", "Bluetooth", "Zigbee", "Thread", "Security", "Kismet", "Wireshark"]
categories: ["Tutorials"]

images: [ "/images/sniffing_invisible_signals_btle_zigbee.png", "/images/nRF_devices.jpeg", "/images/Kismet_BTLE.png", "/images/Wireshark_BTLE_siffer_device_selection.png", "/images/Wireshark_BTLE_sniffing_in_action.png", "/images/Wireshark_802154_siffer_device_selection.png" ]

featuredImage: "/images/sniffing_invisible_signals_btle_zigbee.png"
featuredImagePreview: "/images/sniffing_invisible_signals_btle_zigbee.png"

toc:
  enable: true
seo:
  images: [ "/images/sniffing_invisible_signals_btle_zigbee.png", "/images/nRF_devices.jpeg", "/images/Kismet_BTLE.png", "/images/Wireshark_BTLE_siffer_device_selection.png", "/images/Wireshark_BTLE_sniffing_in_action.png", "/images/Wireshark_802154_siffer_device_selection.png" ]

---

In meinen vorherigen Beiträgen haben wir uns mit dem Prozess [Bridging the Gap: Modernes Kismet auf OpenBSD bringen](/modern-kismet-on-openbsd/) beschäftigt und hatten ein wenig [Spaß mit RTL-SDR auf OpenBSD](/fun-with-rtl-sdr/), um Flugzeuge und Temperatursensoren zu verfolgen. Heute tauche ich tiefer in die unsichtbaren Signale um uns herum ein und verlagere den Fokus auf **Bluetooth Low Energy (BTLE)** und **Zigbee**.

Ich war schon immer neugierig darauf, was IoT-Geräte in meiner Umgebung eigentlich "hinter meinem Rücken" so sagen - Fitnesstracker, smarte Glühbirnen und sogar Zahnbürsten "bewerben" (advertising) ständig ihre Anwesenheit. In diesem Beitrag schauen wir uns an, wie man eine professionelle Sniffing-Umgebung auf **OpenBSD** einrichtet, indem wir sowohl den klassischen [Adafruit Bluefruit Sniffer (nRF51822)](https://www.adafruit.com/product/2269) als auch den modernen [nRF52840 (speziell den nice!nano)](https://nicekeyboards.com/docs/nice-nano/) verwenden.

---

## Die Protokolle: BTLE vs. Zigbee

Bevor wir beginnen, ist es wichtig, die Landschaft des 2,4-GHz-Bands zu verstehen. Wir schauen uns zwei Hauptakteure an: **Bluetooth** und **Zigbee**.

### 1. Bluetooth: Classic vs. Low Energy (BTLE)

Es ist entscheidend, zwischen "normalem" (Classic) Bluetooth und BTLE zu unterscheiden. Es handelt sich um zwei verschiedene Protokolle, die zwar den gleichen Namen und das gleiche Frequenzband teilen, aber nicht zueinander kompatibel sind.

* **Bluetooth Classic:** Entwickelt für kontinuierliche Verbindungen mit hoher Datenrate (wie bei kabellosen Kopfhörern). Es ist stromhungrig und bleibt "verbunden", um den Durchsatz aufrechtzuerhalten.
* **BTLE (Bluetooth Low Energy):** Entwickelt für geringen Stromverbrauch und kurze Datenpakete. Es ermöglicht Geräten, jahrelang mit einer einzigen Knopfzellenbatterie zu laufen. Es verwendet primär eine **Stern-Topologie**, bei der ein zentrales Gerät (wie dein Handy) mit mehreren Peripheriegeräten (wie einem Herzfrequenzmesser) kommuniziert.

#### BTLE-Versionen
* **BTLE v4.x:** Das Fundament des modernen IoT. Es machte den Begriff "Low Energy" massentauglich. Es ist auf 1 Mbit/s begrenzt und hat eine geringere Reichweite.
* **BTLE v5.x:** Der aktuelle Standard. Er führte Geschwindigkeiten von **2 Mbit/s** für schnellere Datenübertragung und **Coded PHY** für eine deutlich höhere Reichweite ein (bis zu 4-fache Distanz).

### 2. Zigbee & Thread (IEEE 802.15.4)

Zigbee und Thread sind die "industriellen Cousins" von BTLE. Während Bluetooth für Personal Area Networks gebaut ist, sind diese Protokolle für Zuverlässigkeit und Skalierbarkeit in der Hausautomation konzipiert.

Ihr Killer-Feature ist die **Mesh-Topologie**, die es jedem netzbetriebenen Gerät (wie einer Glühbirne) ermöglicht, als Repeater zu fungieren und Signale an den nächsten Knoten weiterzuleiten. Dadurch entsteht ein selbstheilendes Netzwerk, das ein ganzes Gebäude abdecken kann, ohne dass jedes Gerät in Reichweite des zentralen Hubs sein muss.

* **Zigbee:** Der etablierte Veteran. Es ist ein Full-Stack-Protokoll, das alles definiert – vom Funkverkehr bis hin zur Struktur eines "Glühbirnen"-Objekts. Es wird häufig in Philips Hue- und IKEA TRÅDFRI-Systemen verwendet.
* **Thread:** Der moderne Newcomer. Obwohl es die gleiche Funkebene (802.15.4) wie Zigbee nutzt, ist Thread **IP-basiert** (IPv6). Das bedeutet, dass Thread-Geräte direkt über das Internet adressiert werden können (via Border Router) und das primäre Netzwerkfundament für **Matter** bilden.

* **Kanäle:** Im Gegensatz zum Frequenzhopping von Bluetooth bleiben 802.15.4-Protokolle meist auf einem **festen Kanal** (nummeriert 11-26).
* **Modulation:** Verwendet O-QPSK mit DSSS (Direct Sequence Spread Spectrum), was es extrem widerstandsfähig gegen Hintergrundrauschen und Störungen durch WLAN macht.

---

### Vergleich auf einen Blick

| Feature | BTLE | Zigbee | Thread |
| :--- | :--- | :--- | :--- |
| **Standard** | Bluetooth SIG | IEEE 802.15.4 | IEEE 802.15.4 (6LoWPAN) |
| **Topologie** | Stern | Mesh | Mesh |
| **Frequenz** | 2.4 GHz (Hopping) | 2.4 GHz (Fest) | 2.4 GHz (Fest) |
| **IP-Support** | Nein (Nativ) | Nein | **Ja (Nativ IPv6)** |
| **Max. Speed** | 2 Mbps (v5) | 250 kbps | 250 kbps |
| **Philosophie** | "Verbinde dich mit dem Handy" | "Baue ein stabiles Netz" | "Jedes Gerät ist im Internet" |

### Weiterführende Literatur

**Für Bluetooth:**
* [Bluetooth Core Specifications](https://www.bluetooth.com/specifications/specs/): Die definitiven (und gewaltigen) technischen Dokumente für jede Version.
* [Bluetooth LE Primer](https://www.bluetooth.com/bluetooth-le-primer/): Ein guter Einstiegspunkt für Entwickler, um den Stack zu verstehen, ohne 3.000 Seiten Spezifikationen lesen zu müssen.

**Für Zigbee & 802.15.4:**
* [Connectivity Standards Alliance (Zigbee)](https://csa-iot.org/all-solutions/zigbee/): Die offizielle Heimat des Zigbee-Standards und des neuen Matter-Protokolls.
* [Digi Knowledge Base: 802.15.4 vs Zigbee](https://www.digi.com/resources/standards-and-technologies/802-15-4-vs-zigbee): Eine fantastische technische Aufschlüsselung der Ebenen zwischen dem Funkstandard und dem Zigbee-Protokoll.
* [Oscium: Zigbee and Wi-Fi Coexistence](https://www.oscium.com/training/zigbee-wifi-coexistence/): Pflichtlektüre, um zu verstehen, wie man Zigbee-Kanäle wählt, die nicht mit dem heimischen WLAN kollidieren.
* [Zigbee vs. Thread](https://www.smarthomeperfected.com/zigbee-vs-thread/): Ein Vergleich zwischen den beiden Hausautomations-Protokollen.
* [Using Wireshark to troubleshoot Thread networks](https://blog.golioth.io/using-wireshark-to-troubleshoot-thread-networks/): Ein praktischer Leitfaden zum Sniffing des aufstrebenden Matter/Thread-Ökosystems.

## Die Hardware

{{< figure
    src="/images/nRF_devices.jpeg"
    alt="Adafruit Bluefruits und einige nice!nanos mit gelöteten Pins"
    caption="Adafruit Bluefruits und einige nice!nanos mit gelöteten Pins"
    class="ma0 w-75"
>}}

### 1. Adafruit Bluefruit Sniffer (nRF51822)

Dieses Gerät basiert auf dem älteren Nordic **nRF51822** Chipsatz. Man findet sie für etwa 30-40 Euro auf **[eBay](https://www.ebay.de/sch/i.html?_nkw=Adafruit+Bluefruit+nRF51822&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF51822&toolid=10001&mkevt=1)**. Es ist ein fantastisches, kompaktes Einsteiger-Tool. Obwohl es die meisten modernen Geräte noch "sehen" kann – da selbst Bluetooth-5-Geräte ihre Anwesenheit meist über Legacy-v4-kompatible Pakete ankündigen – hat es eine klare Grenze: Es kann Verbindungen nicht folgen, die in den schnelleren 2-Mbit/s-Modus wechseln, und es ist blind für die modernen "Extended Advertising"- oder "Long Range"-Pakete, die mit v5 eingeführt wurden.

**Die Identitätskrise: "Friend" vs. "Sniffer":**
Wenn man sowohl den Bluefruit LE Friend als auch den Bluefruit Sniffer besitzt, stellt man schnell fest, dass sie physisch identisch sind. Selbst `dmesg` hilft nicht bei der Unterscheidung, da beide die gleiche USB-zu-UART-Bridge verwenden:

{{< highlight text >}}
uslcom0 at uhub5 port 4 configuration 1 interface 0 "Silicon Labs CP2102N USB to UART Bridge Controller" rev 2.00/1.00 addr 3
ucom0 at uslcom0 portno 0: usb1.1.00004.0
{{< /highlight >}}

**Der Blink-Test:** Achte beim Einstecken auf die LEDs. Die **Sniffer**-Firmware blinkt **Blau**, während die **Friend**-Firmware **Rot** blinkt.

### 2. nRF52840 (Das moderne Kraftpaket)
Der **nRF52840** ist der Nachfolger der nRF51-Serie und unterstützt die volle **BTLE v5**-Suite sowie den **IEEE 802.15.4**-Standard. Dies macht ihn zu einem "universellen" IoT-Sniffer, der alles von High-Speed Bluetooth 5 bis hin zu Mesh-Netzwerken wie **Zigbee** und **Thread** mithören kann.

Ich verwende den **nice!nano**, ein ProMicro-kompatibles Board, das oft in maßgeschneiderten mechanischen Tastaturen zum Einsatz kommt. Es ist perfekt für diese Aufgabe, da es auf ein Standard-Breadboard passt und ein eingebautes Ladegerät besitzt, falls man sein Sniffing-Equipment mobil machen möchte.

{{< highlight text >}}
umodem0 at uhub5 port 1 configuration 1 interface 0 "Adafruit Feather nRF52840 Express" rev 2.00/1.00 addr 3
umodem0: data interface 1, has no CM over data, has break
umodem0: status change notification available
ucom0 at umodem0: usb1.1.00001.1
{{< /highlight >}}

Diese nice!nano-Boards sind ebenfalls auf [eBay](https://www.ebay.de/sch/i.html?_nkw=nRF52840+Nice!Nano&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF52840&toolid=10001&mkevt=1) für oft weniger als 10 Euro zu finden (Nachbauten teils ab 4 Euro). Bei diesem Preis lohnt es sich, eine kleine Handvoll davon zu kaufen. So kann man ein Board fest für jedes Protokoll (BTLE oder 802.15.4) reservieren und spart sich das ständige Umflashen.

Die Dokumentation findest du [hier](https://docs.nordicsemi.com/bundle/ncs-latest/page/zephyr/boards/others/promicro_nrf52840/doc/index.html) und die [Pinbelegung/Schaltpläne hier](https://nicekeyboards.com/docs/nice-nano/pinout-schematic).

#### Flashing des nice!nano
Um das Board in einen Sniffer zu verwandeln, nutzen wir die Firmware aus dem [Makerdiary nRF52840 ConnectKit](https://github.com/makerdiary/nrf52840-connectkit) Repository.

1.  **Repository klonen:**
    {{< highlight bash >}}
    git clone https://github.com/makerdiary/nrf52840-connectkit.git
    cd nrf52840-connectkit
    {{< /highlight >}}

2.  **Bootloader-Modus aktivieren:** Verbinde das Gerät via USB. Benutze eine Pinzette, um die Pins **RST** und **GND** zweimal schnell kurzzuschließen. Alternativ können natürlich auch die Pins angelötet werden, und dann mit einem normalen Jumper RST und GND überbrückt werden.
3.  **Laufwerk einbinden:**
    {{< highlight bash >}}
    doas mount /dev/sd1i /mnt  # Gerätenamen ggf. anpassen
    {{< /highlight >}}
4.  **Flashen:** Wähle dein Zielprotokoll:
    * **Für BTLE:** `doas cp ./firmware/ble_sniffer/nrf_sniffer_for_bluetooth_le_v4.1.1.uf2 /mnt/`

    {{< highlight bash >}}
    umodem0 at uhub5 port 3 configuration 1 interface 0 "ZEPHYR nRF Sniffer for Bluetooth LE" rev 2.00/2.04 addr 3
    umodem0: data interface 1, has CM over data, has no break
    umodem0: status change notification available
    ucom0 at umodem0: usb1.1.00003.1
    {{< /highlight >}}

    * **Für Zigbee:** `doas cp ./firmware/nrf802154_sniffer/nrf802154_sniffer_v0.7.2.uf2 /mnt/`

    {{< highlight bash >}}
    umodem0 at uhub5 port 3 configuration 1 interface 0 "Nordic Semiconductor ASA nRF 802154 Sniffer" rev 2.00/3.06 addr 3
    umodem0: data interface 1, has CM over data, has no break
    umodem0: status change notification available
    ucom0 at umodem0: usb1.1.00003.1
    {{< /highlight >}}

5.  **Verifizieren:** Überprüfe `dmesg`. Es sollte entweder als "ZEPHYR nRF Sniffer for Bluetooth LE" oder "Nordic Semiconductor ASA nRF 802154 Sniffer" erscheinen.

{{< admonition type="warning" title="Originale Firmware wiederherstellen" open=true >}}
Wenn du die ursprüngliche Firmware wiederherstellen möchtest, lade die Datei **s140_nrf52_6.1.1_softdevice.uf2** aus dem offiziellen Repository herunter: [Adafruit nRF52 Bootloader Releases](https://github.com/adafruit/Adafruit_nRF52_Bootloader/releases/tag/softdevice-uf2).
{{< /admonition >}}

---

## Software-Einrichtung auf OpenBSD

Wir verwenden **Kismet** für die Geräteerkennung in Echtzeit und **Wireshark** für die tiefergehende Paketanalyse.

{{< admonition type="info" title="OpenBSD Ports verfügbar" open=true >}}
OpenBSD bietet native Ports und Pakete für diese Tools an, was die Installation unkompliziert und gut in das Basissystem integriert macht.
{{< /admonition >}}

### 1. Kismet installieren
{{< highlight bash >}}
doas pkg_add kismet
doas usermod -G _kismet dein_benutzername
{{< /highlight >}}

### 2. Wireshark & Extcap Plugins
Wir benötigen die Python-Abhängigkeiten und die Plugins aus dem gerade geklonten Git-Repo.

{{< highlight bash >}}
doas pkg_add wireshark py3-serial py3-psutil
doas usermod -G _wireshark dein_benutzername
mkdir -p ~/.local/lib/wireshark/extcap
{{< /highlight >}}

**BTLE Plugin installieren:**
{{< highlight bash >}}
cp -r tools/ble_sniffer/extcap/* ~/.local/lib/wireshark/extcap/
chmod +x ~/.local/lib/wireshark/extcap/nrf_sniffer_ble.*
{{< /highlight >}}

**Zigbee (802.15.4) Plugin installieren:**
{{< highlight bash >}}
cp tools/nrf802154_sniffer/nrf802154_sniffer/* ~/.local/lib/wireshark/extcap/nrf802154_sniffer.*
chmod +x ~/.local/lib/wireshark/extcap/nrf802154_sniffer.*
{{< /highlight >}}

{{< admonition type="warning" title="OpenBSD Serial Port Patch" open=true >}}
Auf OpenBSD erkennt `py-serial` den nRF-Sniffer oft nicht automatisch. Um dies zu beheben, muss der folgende Patch auf die Datei `nrf802154_sniffer.py` angewendet werden.

**Hinweis:** Dieser Patch geht davon aus, dass dein Gerät unter `/dev/cuaU0` erreichbar ist. Falls dein Sniffer an einem anderen Port hängt, musst du das Skript entsprechend anpassen.
{{< /admonition >}}

{{< highlight bash >}}
diff --git a/tools/nrf802154_sniffer/nrf802154_sniffer/nrf802154_sniffer.py b/tools/nrf802154_sniffer/nrf802154_sniffer/nrf802154_sniffer.py
index afb7198..2e15ffe 100755
--- a/tools/nrf802154_sniffer/nrf802154_sniffer/nrf802154_sniffer.py
+++ b/tools/nrf802154_sniffer/nrf802154_sniffer/nrf802154_sniffer.py
@@ -78,7 +78,7 @@ class Nrf802154Sniffer(object):
      CTRL_ARG_LOGGER = 6
 
      # Pattern for packets being printed over serial.
-    RCV_REGEX = 'received:\s+([0-9a-fA-F]+)\s+power:\s+(-?\d+)\s+lqi:\s+(\d+)\s+time:\s+(-?\d+)'
+    RCV_REGEX = r'received:\s+([0-9a-fA-F]+)\s+power:\s+(-?\d+)\s+lqi:\s+(\d+)\s+time:\s+(-?\d+)'
 
      TIMER_MAX = 2**32
 
@@ -187,6 +187,7 @@ class Nrf802154Sniffer(object):
          """
          res = []
          res.append("extcap {version=0.7.2}{help=https://github.com/NordicSemiconductor/nRF-Sniffer-for-802.15.4}{display=nRF Sniffer for 802.15.4}")
+        res.append ("interface {value=/dev/cuaU0}{display=nRF Sniffer for 802.15.4}")
          for port in comports():
              if port.vid == Nrf802154Sniffer.NORDICSEMI_VID and port.pid == Nrf802154Sniffer.SNIFFER_802154_PID:
                  res.append ("interface {value=%s}{display=nRF Sniffer for 802.15.4}" % (port.device,) )
{{< /highlight >}}

### Geräteberechtigungen & Automatisierung

Während **Kismet** SUID-Root-Helfer verwendet und "einfach so" funktioniert, benötigt **Wireshark** Zugriff auf die `/dev/cuaU*` Geräte als Mitglied der `_wireshark`-Gruppe.

Auf OpenBSD lässt sich dies mit **[hotplugd(8)](https://man.openbsd.org/hotplugd)** automatisieren. So werden die Berechtigungen sofort angepasst, wenn du den Sniffer einsteckst.

#### 1. Sniffer identifizieren
Führe `usbdevs -v` aus, um die Seriennummern deiner nRF-Geräte zu finden. Diese benötigst du für die Skripte.

#### 2. Das Attach-Skript erstellen
Erstelle `/etc/hotplugd/attach` (und mache es mit `chmod +x` ausführbar). Dieses Skript prüft auf deine Hardware und ordnet den Treiber (z.B. `uslcom0`) dem passenden Geräteknoten zu.

{{< highlight bash >}}
#!/bin/sh

DEVCLASS=${1}
DEVNAME=${2}
# Konfiguration: Liste der autorisierten Seriennummern
SNIFFERS_SERIAL="DEINE_SERIENNUMMERN_HIER"

case ${DEVCLASS} in
0)
    # Wir agieren nur auf dem Hardware-Treiber (z.B. uslcom0)
    # Das verhindert Race Conditions mit 'ucom'.
    sleep 1
    for serial in $SNIFFERS_SERIAL; do
        is_match=$(usbdevs -v | grep -B 5 "driver: $DEVNAME" | grep "$serial")

        if [ -n "$is_match" ]; then
            # Unit-Nummer extrahieren: uslcom0 -> 0 -> /dev/cuaU0
            UNIT=$(echo "$DEVNAME" | tr -d -c '0-9')
            CUADEV="/dev/cuaU$UNIT"

            if [ -c "$CUADEV" ]; then
                GROUP=_wireshark
                chgrp "$GROUP" "$CUADEV"
                chmod 660 "$CUADEV"
                # Statusdatei für Detach-Tracking erstellen
                touch "/var/run/hotplug_is_sniffer_$DEVNAME"
                logger -t hotplugd "ERFOLG: Hardware $DEVNAME ($serial) auf $CUADEV gemappt, Gruppe auf $GROUP geändert"
                break
            fi
        fi
    done
    ;;
esac
{{< /highlight >}}

#### 3. Das Detach-Skript erstellen
Erstelle `/etc/hotplugd/detach` (ebenfalls `chmod +x`), um den Ursprungszustand beim Abziehen wiederherzustellen.

{{< highlight bash >}}
#!/bin/sh

DEVCLASS=${1}
DEVNAME=${2}
STATE_FILE="/var/run/hotplug_is_sniffer_${DEVNAME}"

case ${DEVCLASS} in
0)
    if [ -f "$STATE_FILE" ]; then
        UNIT=$(echo "$DEVNAME" | tr -d -c '0-9')
        CUADEV="/dev/cuaU$UNIT"

        if [ -c "$CUADEV" ]; then
            chown root:dialer "$CUADEV"
            chmod 660 "$CUADEV"
            logger -t hotplugd "$CUADEV nach Detach auf root:dialer zurückgesetzt."
        fi
        rm "$STATE_FILE"
    fi
    ;;
esac
{{< /highlight >}}

{{< admonition type="warning" title="Einschränkungen von hotplugd" open=true >}}
`hotplugd` ist nicht perfekt. Trotz Detach-Skript kann es vorkommen, dass `/dev/cuaU*`-Geräte der Gruppe `_wireshark` zugeordnet bleiben – etwa wenn der Rechner heruntergefahren wird, während das Gerät noch eingesteckt ist.
{{< /admonition >}}

## Sniffing in Aktion

{{< admonition type="note" title="Hardware-Einschränkungen" open=true >}}
Bitte beachte, dass ich derzeit keine aktiven **Zigbee**- oder **Thread**-Geräte in meiner Umgebung habe. Daher kann ich momentan keine Live-Screenshots von erkannten Geräten für diese Protokolle zeigen.
{{< /admonition >}}

### Nutzung mit Kismet
Stecke dein Gerät ein und starte Kismet mit Verweis auf das serielle Gerät (meist `/dev/cuaU0`).

**Für BTLE:**
{{< highlight bash >}}
kismet -c nrf51822:type=nrf51822,device=/dev/cuaU0,name=btle_sniffer
{{< /highlight >}}

Die Kismet-Weboberfläche erreichst du unter `http://localhost:2501`.

{{< figure
    src="/images/Kismet_BTLE.png"
    alt="Kismet bei der BTLE-Gerätesuche"
    caption="Kismet bei der BTLE-Gerätesuche"
    class="ma0 w-75"
>}}

**Für Zigbee (802.15.4):**
{{< highlight bash >}}
kismet -c nrf52840:type=nrf52840,device=/dev/cuaU0,name=zigbee_sniffer,channel_hop=false,channel=11
{{< /highlight >}}
*Hinweis: Zigbee bleibt auf einem festen Kanal. 11, 15, 20 und 25 sind gängig für Smart-Home-Hubs.*

### Nutzung mit Wireshark
Wenn die Plugins korrekt installiert sind, siehst du "nRF Sniffer for Bluetooth LE" oder "nRF Sniffer for 802.15.4" in der Interface-Liste.

#### BTLE Sniffing

{{< figure
    src="/images/Wireshark_BTLE_siffer_device_selection.png"
    alt="BTLE Sniffer in Wireshark auswählen"
    caption="BTLE Sniffer in Wireshark auswählen"
    class="ma0 w-75"
>}}

{{< figure
    src="/images/Wireshark_BTLE_sniffing_in_action.png"
    alt="Wireshark BTLE Sniffer in Aktion"
    caption="Wireshark BTLE Sniffer in Aktion"
    class="ma0 w-75"
>}}

#### Zigbee Sniffing

{{< figure
    src="/images/Wireshark_802154_siffer_device_selection.png"
    alt="802154 Sniffer in Wireshark auswählen"
    caption="802154 Sniffer in Wireshark auswählen"
    class="ma0 w-75"
>}}

**Profi-Tipp für Zigbee:** Um verschlüsselten Traffic zu sehen, gehe zu **Preferences > Protocols > Zigbee** und füge den "Trust Center Link Key" hinzu: `5A:69:67:42:65:65:41:6C:6C:69:61:6E:63:65:30:39`. Mehr dazu findest du unter [Sniff Zigbee traffic](https://www.zigbee2mqtt.io/advanced/zigbee/04_sniff_zigbee_traffic.html).

---

## Fazit

Durch die Kombination der sicherheitsorientierten Umgebung von **OpenBSD** mit der Vielseitigkeit des **[nRF52840](https://www.ebay.de/sch/i.html?_nkw=nRF52840+Nice!Nano&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF52840&toolid=10001&mkevt=1)** (und seinem Vorgänger, dem **[nRF51822 Bluefruit Sniffer](https://www.ebay.de/sch/i.html?_nkw=Adafruit+Bluefruit+nRF51822&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF51822&toolid=10001&mkevt=1)**) hast du einen hochwertigen Protokoll-Analyser zur Hand. Vom Tracking von Bluetooth-Advertisements bis hin zur Kartierung von Zigbee-Mesh-Netzwerken ist das 2,4-GHz-Spektrum keine "Black Box" mehr.

Während der [nRF51822](https://www.ebay.de/sch/i.html?_nkw=Adafruit+Bluefruit+nRF51822&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF51822&toolid=10001&mkevt=1) ein Klassiker für reine BLE-Aufgaben bleibt, öffnet die Multi-Protokoll-Unterstützung des [nRF52840](https://www.ebay.de/sch/i.html?_nkw=nRF52840+Nice!Nano&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=nRF52840&toolid=10001&mkevt=1) die Tür zu modernen IoT-Untersuchungen, die früher mit kleinem Budget kaum zugänglich waren.

### Nächste Schritte
Das Setup steht, aber der Äther ist noch ruhig... vorerst. Das ist ein hervorragender Vorwand, um bald tiefer in die **Hausautomation** einzusteigen. Ich plane, diverse Zigbee- und Thread-basierte Sensoren in mein Labor zu integrieren. Sobald diese vorhanden sind, folgt ein detaillierter Artikel, in dem ich zeige, wie diese Geräte in Echtzeit erfasst und analysiert werden. Bleibt dran!
