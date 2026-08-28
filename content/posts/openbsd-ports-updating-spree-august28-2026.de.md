---
title: "Frischer Wind in den OpenBSD Ports: SDR, Pentesting-Tools, OpenVOX und Plaso"
subtitle: "Ein Rückblick auf die jüngsten Ports-Updates, Frequenzwechsel und Forensik-Hürden"
description: "Ein detaillierter Einblick in die neuesten OpenBSD-Ports-Updates: Von SDR-Importen wie Gqrx und ReadsB über Security-Tools bis hin zum kniffligen Plaso- und Libyal-Upgrade."
date: 2026-08-28T22:00:00+02:00
draft: false
license: "MIT"
tags: ["OpenBSD", "Ports", "SDR", "Gqrx", "Plaso", "OpenVOX", "Security"]
categories: ["OpenBSD", "Tech", "Porting"]
images: [ "/images/ports-sprint-august-2026.jpg", "/images/Gqrx.png" ]

featuredImage: "/images/ports-sprint-august-2026.jpg"
featuredImagePreview: "/images/ports-sprint-august-2026.jpg"

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
  images: [ "/images/ports-sprint-august-2026.jpg", "/images/Gqrx.png" ]
---

Es wurde mal wieder Zeit: Der [letzte größere Upgrade-Sprint](/openbsd-ports-updating-spree-june4/) in den OpenBSD-Ports lag bereits eine Weile zurück, und auf meiner To-Do-Liste hatten sich etliche Pakete angesammelt, die dringend etwas Zuneigung brauchten. Wer das OpenBSD-Ports-Ökosystem kennt, weiß, dass ein solches Update selten nur aus dem Anpassen einer `Makefile`-Versionsnummer besteht. Es ist vielmehr eine muntere Entdeckungsreise durch Abhängigkeitsbäume, Build und Test Changes, Build Fehler, etc.

In den vergangenen Tagen und Wochen habe ich mich durch verschiedene Themengebiete gearbeitet: Von Software Defined Radio (SDR) über Pentesting- und Web-Security-Tools bis hin zur Infrastruktur-Automatisierung mit OpenVOX und der digitalen Forensik rund um Plaso. Hier ist ein Überblick über die Arbeit und die Geschichten hinter den Commits.

---

## Frequenzen, Flugzeuge und Frequenzwechsel: Der SDR-Bereich

Der SDR-Bereich hat ein ordentliches Upgrade erfahren. Besonders gefreut habe ich mich über den Import von **`comms/gqrx`** samt der benötigten Abhängigkeit **`comms/gr-osmosdr`**. Gqrx ist ein extrem populärer, Open-Source-basierter SDR-Empfänger, der auf GNU Radio und dem Qt-GUI-Toolkit aufbaut. Damit steht unter OpenBSD nun eine weitere hervorragende grafische Oberfläche zur Erkundung des Äther-Spektrums bereit.

{{< figure
    src="/images/Gqrx.png"
    alt="Gqrx im Einsatz"
    caption="Gqrx im Einsatz"
    class="ma0 w-75"
>}}

Ein wichtiger Tipp kam von `jbg@`, der mich auf **`comms/readsb`** aufmerksam machte. `readsb` ist ein performanter Mode-S/ADS-B/TIS-Decoder für Geräte wie den RTL-SDR, HackRF oder Mode-S Beast.

Allerdings hatte `readsb` beim Portieren so einige Build-Probleme im Gepäck, gebaut und getestet mehr oder weniger nur für Linux. Es benötigte etliche Patches an verschiedenen Ecken, den Code auf OpenBSD sauber zum Bauen und Laufen zu bringen. Es hat definitiv eine Weile gedauert, die Inkompatibilitäten zu überwinden – aber die Arbeit hat sich mehr als gelohnt!

{{< admonition type="tip" title="Praxis-Tipp: ReadsB & Dump1090 kombinieren" open=true >}}
In meinen Tests fing `readsb` zuverlässig mehr Flugzeuge ab, zumindest mit einem RTL-SDR Blog, als das bisherige `dump1090` in den Ports. Der einzige Haken: `readsb` bringt von Haus aus kein eigenes Web-Interface zur Kartendarstellung mit. Die Lösung ist jedoch elegant:
1. `dump1090` im **Netzwerk-Only-Modus** starten.
2. `readsb` so konfigurieren, dass es die Flugdaten via **Beast-Protokoll** an `dump1090` weiterleitet.

Ergebnis: Die überlegene Dekodierleistung von `readsb` gepaart mit der hübschen Kartenansicht von `dump1090`!
{{< /admonition >}}


Wer selbst in die Welt der Signale einsteigen möchte: Zuverlässige [RTL-SDR Blogs gibt es bei eBay](https://ebay.us/XTunjd#affiliate) als ideale Hardware-Basis.

### Stabilität unter der Haube

Neben neuen Ports gab es auch wichtige Stabilitäts-Fixes. **SASANO Takayoshi (`uaa@`)** hat in `comms/rtl-sdr` und `comms/soapy-rtlsdr` wichtige Korrekturen vorgenommen: Der Wechsel von asynchronen auf synchrone Aufrufe sorgt nun dafür, dass das Umschalten von Frequenzen ohne lästige Hänger oder Abstürze funktioniert.

**Übersicht SDR-Updates:**
* `comms/gr-osmosdr` *(Neu importiert)*
* `comms/gqrx` *(Neu importiert)*
* `comms/readsb` *(Neu importiert)*
* `comms/liquid-dsp`: Update 1.7.0 -> 1.8.2
* `comms/rtl-sdr` & `comms/soapy-rtlsdr` *(Stabilitäts-Fixes)*

Mehr zu RTL-SDR in meinem früheren Blog [hier](/fun-with-rtl-sdr/).

---

## Entwicklung & Pentesting-Tools

Weiter ging es im Bereich der Sicherheits- und Entwicklungs-Tools. Das beliebte Reverse-Engineering-Werkzeug `devel/apktool` wurde auf Version 3.0.3 angehoben, und auch `security/exploitdb` hat einen frischen Datensatz erhalten, sowie `security/py-fickling` ein kleines Update erhielt.

Rund um das WordPress-Sicherheits-Tool `wpscan` (Update auf 4.1.0) erlaubte es auch eine Anzahl an Abhängigkeiten upzudaten:

{{< highlight yaml >}}
security/wpscan:                4.0.0     -> 4.1.0
devel/ruby-activesupport:       8.1.3     -> 8.1.3.1
www/ruby-ethon:                 0.16.0    -> 0.18.0
www/ruby-typhoeus:              1.4.1     -> 1.6.0
www/ruby-ferrum:                0.17.2    -> 0.18.0
security/py-fickling:           0.1.11    -> 0.1.12
security/exploitdb:             2026-06-02 -> 2026-08-19
{{< /highlight >}}

Wenn man schon mal dabei ist, räumt man auch die Nebenschauplätze auf: Obwohl keine direkten internen Abhängigkeiten bestanden, wurden `archivers/ruby-rubyzip` (auf 3.5.0) und `devel/ruby-zeitwerk` (auf 2.8.3) gleich mit aktualisiert.

---

## Infrastruktur: OpenVOX & Puppetboard

Auch im OpenVOX (formerly known as Puppet) Ökosystem gab es Bewegung. Neben diversen Ruby-Bibliotheken wurde die Server-Komponente `sysutils/openvox-server/8` von Version 8.14.0 auf 8.15.2 gebracht. Für das Monitoring-Frontend `puppetboard` wurden zudem dessen Abhängigkeiten `py-cachelib` (0.17.0) und `py-flask-caching` (2.5.0) aktualisiert.

* `net/ruby-msgpack`: 1.8.1 $\rightarrow$ 1.8.4
* `www/ruby-faraday`: 2.14.2 $\rightarrow$ 2.14.3
* `sysutils/ruby-openvoxserver-ca`: 3.2.0 $\rightarrow$ 3.3.0
* `sysutils/openvox-server/8`: 8.14.0 $\rightarrow$ 8.15.2

---

## Forensik mit Plaso & die Libyal-Achterbahn

Der zeitintensivste Teil dieses Sprints war das Update der Digital-Forensics-Kette rund um **Plaso** (auf Version 20260720, siehe auch die offiziellen [Plaso Release Notes](https://osdfir.blogspot.com/2026/07/plaso-20260720-released.html)).

Plaso bringt ein massives Geflecht an `libyal`-Bibliotheken mit sich. Die Gelegenheit habe ich direkt genutzt, um Ordnung im Port-Baum zu schaffen und `sysutils/libfsntfs` konsequent nach `sysutils/libyal/libfsntfs` zu verschieben.

{{< admonition type="note" title="Stolpersteine beim Bauen: Autoconf & FUSE" open=true >}}
Beim Bauen der aktualisierten `libyal`-Ports lief ich gleich in zwei interessante Probleme:

1. **Autoconf-Verwirrung:** Die `libyal`-Suiten haben ihre Test-Generierung auf `autoconf` umgestellt. Nach anfänglichen Fehlern und einem kurzen, jedoch sehr hilfreichen Austausch mit dem Upstream-Maintainer **Joachim Metz** auf GitHub stellte sich heraus: Ich nutzte schlicht die falsche Autoconf-Version. Mit der korrekten Version lief der Prozess wie am Schnürchen.
2. **FUSE-Inkompatibilität:** Sämtliche `libyal`-Dateisystem-Ports (`libfsfat`, `libfsxfs`, etc.) scheiterten an der Funktion `fuse_unmount()`. Der Hintergrund: OpenBSD nutzt im Basissystem eine ältere FUSE 2.X-Schnittstelle. `libyal` ging jedoch davon aus, dass bereits FUSE 3.X vorliegt. Das Erstellen passender Patches löste das Problem. Auch ist der Maintainer informiert, der entsprechend alle `libfsXXX` entsprechend anpassen wird.
{{< /admonition >}}

Dank der hervorragenden und umfangreichen Test-Suites, nachdem die dann endlich mit der richtigen Autoconf Version durchliefen, die Plaso und die `libyal`-Bibliotheken mitbringen, war das finale Testen extrem beruhigend: Wenn hunderte Tests erfolgreich durch das Terminal rauschen, schläft es sich als Porter gleich viel ruhiger.

---

## Fazit und Ausblick

Wie sich gezeigt hat, hatte sich seit dem letzten Sprint einiges angesammelt. Es ist ein gutes Gefühl zu wissen, dass relativ kurz vor dem Release-Freeze für OpenBSD 8.0 der Großteil der von mir betreuten Ports auf dem aktuellsten Stand ist. 

Hoffentlich kommen bis zum Freeze nur noch kleinere Updates hier und da – ganz ohne Stress kurz vor dem Release! Bis dahin: Viel Spaß beim Ausprobieren der neuen Ports und Happy Hacking!

