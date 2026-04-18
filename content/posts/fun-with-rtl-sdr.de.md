---
title: "Spaß mit RTL-SDR auf OpenBSD: Von Flugzeugen, Funk-Thermometern und Radio"
subtitle: "Pakete schnappen direkt aus der Luft – auf OpenBSD."
date: 2026-04-17T21:36:52+02:00
lastmod: 2026-04-17T21:36:52+02:00
draft: false
author: ""
authorLink: ""
description: "Vom Flugzeug-Tracking bis zum Auslesen der smarten Wetterstation vom Nachbarn: Entdecke die Welt von SDR auf OpenBSD. Einfach nur Signale ohne Schnörkel."
license: "MIT"
images: [ "/images/fun-with-rtl-sdr.png", "/images/RTL-SDR.jpeg", "/images/dump1090.png", "/images/kismet_rtl433.png", "/images/kismet_rtladsb.png" ]

tags: ["OpenBSD", "SDR", "RTL-SDR", "Radio", "Kismet"]
categories: ["Tech", "Networking"]

featuredImage: "/images/fun-with-rtl-sdr.png"
featuredImagePreview: "/images/fun-with-rtl-sdr.png"

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
  images: [ "/images/fun-with-rtl-sdr.png", "/images/RTL-SDR.jpeg", "/images/dump1090.png", "/images/kismet_rtl433.png", "/images/kisme
t_rtladsb.png" ]
---

Nach meinem letzten Post über [Modernes Kismet auf OpenBSD](/modern-kismet-on-openbsd/) wird es Zeit, dass wir uns die Hardware anschauen, die den ganzen Funk-Spaß erst möglich macht. Wenn du noch einen USB-Port frei hast und dir für rund 50€ einen RTL-SDR-Dongle besorgst, verwandelst du deine OpenBSD-Kiste in einen fähigen Radio-Scanner.

## Was ist eigentlich SDR?

**Software Defined Radio (SDR)** verlagert den Prozess der Signalverarbeitung von spezialisierten Hardwareschaltkreisen direkt in die Software. Früher war es so: Wolltest du eine neue Frequenz hören oder ein neues Protokoll knacken, musstest du ein komplett neues Funkgerät kaufen. Bei SDR ist die Hardware eigentlich nur ein "dummes" Bauteil, das ein Stück des Frequenzspektrums digitalisiert – dein Prozessor übernimmt dann das eigentliche Rechnen, Filtern und Dekodieren.

Der **[RTL-SDR](https://www.rtl-sdr.com/)** ist dabei quasi die Einstiegsdroge. Eigentlich waren das mal billige USB-Sticks für digitales Fernsehen (DVB-T), aber findige Hacker haben gemerkt, dass man den Realtek RTL2832U-Chip in einen Rohdaten-Modus versetzen kann. Damit kann man so ziemlich alles zwischen **500 kHz und 1,7 GHz** empfangen.

Die Klassiker wie den RTL-SDR Blog V3 (oder den neuen V4) sowie passende Antennen bekommt man problemlos auf [eBay](https://www.ebay.de/sch/i.html?_nkw=RTL+SDR+BLOG&_sacat=0&_from=R40&_trksid=p2334524.m570.l1313&LH_TitleDesc=0&_odkw=RTL+SDR&_osacat=0&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=RTLSDR&toolid=10001&mkevt=1).

Für diesen Artikel habe ich meinen treuen RTL-SDR Blog V3 benutzt.

{{< figure
    src="/images/RTL-SDR.jpeg"
    alt="RTL-SDR Blog V3"
    caption="Das Arbeitstier: RTL-SDR Blog V3"
    class="ma0 w-75"
>}}

## Das OpenBSD-Setup

Unter OpenBSD werfen wir zuerst einen Blick in die Ausgabe von `dmesg`, um zu sehen, ob das Teil als USB-Gerät erkannt wird:

{{< highlight shell "linenos=table" >}}
ugen2 at uhub5 port 4 "Realtek RTL2838UHIDIR" rev 2.00/1.00 addr 4
{{< /highlight >}}

Das sieht gut aus. Wenn das Paket `comms/rtl-sdr` installiert ist, können wir mit `rtl_test` kurz die Lage checken:

{{< highlight shell "linenos=table" >}}
sudo rtl_test
Found 1 device(s):
  0:  Realtek, RTL2838UHIDIR, SN: 00000001

Using device 0: Generic RTL2832U OEM
Found Rafael Micro R820T tuner
...
Reading samples in async mode...
lost at least 76 bytes
...
{{< /highlight >}}

### Ein Wort zum "Async USB"

Wenn du `rtl_test` laufen lässt, siehst du wahrscheinlich diese Meldungen:
`lost at least 76 bytes`, `lost at least 68 bytes`...

Keine Panik! Das liegt daran, dass OpenBSD das asynchrone USB-Protokoll, das die Treiber auf anderen Systemen nutzen, nicht nativ unterstützt. "Async" wird hier quasi über das normale synchrone USB-Protokoll getunnelt. Das führt bei hohen Abtastraten zu winzigen Datenverlusten, aber für normales Monitoring oder digitale Pakete ist das nicht weiter tragisch.

## Spielereien auf der Konsole

### 1. Ganz klassisch: UKW-Radio

Lust auf Musik? Du kannst die Rohdaten von `rtl_fm` direkt in `ffplay` pipen, um deinen Lieblingssender zu hören:

{{< highlight shell "linenos=table" >}}
rtl_fm -f 91.9M -M wfm -s 170k -r 44.1k | ffplay -nodisp -f s16le -ar 44.1k -
{{< /highlight >}}

### 2. Sensoren überwachen mit `rtl_433`

Das Tool `rtl_433` ist das Schweizer Taschenmesser für das 433,92 MHz Band. Damit dekodiert man Signale von Außenthermetern, Reifendrucksensoren oder sogar manchen Stromzählern, und ähnlichem.

{{< highlight shell "linenos=table" >}}
sudo rtl_433 -s 1024k
{{< /highlight >}}

{{< admonition type="warning" title="Kleiner Tipp zu den Sampleraten" >}}
Der RTL-SDR unterstützt theoretisch höhere Sampleraten, aber ich habe gemerkt, dass alles über **1024k** unter OpenBSD mit dem **V3**-Dongle instabil wird und gerne mal abstürzt. Wenn du die neue **V4**-Hardware hast, probier es aus, aber 1024k ist ein super Startwert für ein stabiles System.
{{< /admonition >}}

{{< highlight shell "linenos=table" >}}
sudo rtl_433 -s 1024k
rtl_433 version 25.12 (2025-12-12) inputs file rtl_tcp RTL-SDR with TLS
Reading conf from "/etc/rtl_433/rtl_433.conf".
Found Rafael Micro R820T tuner
[SDR] Using device 0: Realtek, RTL2838UHIDIR, SN: 00000001, "Generic RTL2832U OEM"
[R82XX] PLL not locked!
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _
time      : 2026-04-14 20:59:35
model     : Nexus-TH     House Code: 196
Channel   : 2            Battery   : 1             Temperature: 11.60 C      Humidity  : 62 %
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _
time      : 2026-04-14 20:59:39
model     : Oregon-THGR810                         House Code: 203
Channel   : 0            Battery   : 0             Celsius   : 8.70 C        Humidity  : 72 %
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _
{{< /highlight >}}

In meinem Test hab ich sofort Sensoren aus der Nachbarschaft beobachten können:
* **Nexus-TH:** Temperatur 11,6 °C, 62 % Feuchtigkeit.
* **Oregon-THGR810:** 8,7 °C, 72 % Feuchtigkeit.

Schon witzig zu sehen, wie viele "smarte" Geräte in der eigenen Straße so rumfunken.

### 3. Flugzeug-Tracking mit `dump1090`

Flugzeuge senden ihre Position und Höhe via **ADS-B** auf 1090 MHz. Mit `dump1090` wird dein Terminal zum Radarschirm:

{{< highlight shell "linenos=table" >}}
sudo dump1090
Found 1 device(s):
0: Realtek, RTL2838UHIDIR, SN: 00000001 (currently selected)
Found Rafael Micro R820T tuner
Max available gain is: 49.60
Setting gain to: 49.60
Exact sample rate is: 2000000.052982 Hz
Gain reported by device: 49.60
*5d4cadbbcf5037;
CRC: cf5037 (ok)
DF 11: All Call Reply.
  Capability  : Level 2+3+4 (DF0,4,5,11,20,21,24,code7 - is on airborne)
  ICAO Address: 4cadbb

*8d4cadbb586f963f3468ea17a936;
CRC: 17a936 (ok)
DF 17: ADS-B message.
  Capability     : 5 (Level 2+3+4 (DF0,4,5,11,20,21,24,code7 - is on airborne))
  ICAO Address   : 4cadbb
  Extended Squitter  Type: 11
  Extended Squitter  Sub : 0
  Extended Squitter  Name: Airborne Position (Baro Altitude)
    F flag   : odd
    T flag   : non-UTC
    Altitude : 21225 feet
    Latitude : 73626 (not decoded)
    Longitude: 26858 (not decoded)

*20000db9733fa9;
CRC: 733fa9 (ok)
DF 4: Surveillance, Altitude Reply.
  Flight Status  : Normal, Airborne
  DR             : 0
  UM             : 0
  Altitude       : 21225 feet
  ICAO Address   : 4cadbb
...
{{< /highlight >}}

### Deep Dive: Was bedeuten die "DF"-Codes?

Wenn du dich fragst, was dieser Buchstabensalat wie **DF 17** oder die **ICAO-Adresse** bedeutet: Willkommen in der Welt der Transponder-Technik!

Das **Downlink Format (DF)** sind die ersten 5 Bits einer Nachricht. Sie verraten dem Empfänger, wie er den Rest lesen muss:
* **DF 11**: Der "All Call"-Ruf. Hier meldet sich ein Flieger mit seiner eindeutigen 24-Bit-Kennung.
* **DF 17**: Der "Extended Squitter" – das ist das Goldstück von ADS-B mit GPS-Position, Höhe und Speed.

Wer tiefer in die Bit-Ebene einsteigen will (etwa warum man zwei Pakete braucht, um die Position korrekt zu berechnen), sollte sich diese Seiten mal anschauen:
* **[The 1090 Megahertz Riddle](https://mode-s.org/1090mhz/)**: Ein freies Buch von Junzi Sun, das Mode S and ADS-B Dekodierung sehr gut erklärt.
* **[The Radar Tutorial](https://www.radartutorial.eu/13.ssr/sr24.en.html)**: Alles über Radar und Mode-S-Formate.
* **[RTL-SDR.com](https://www.rtl-sdr.com/)**: Der definitive Blog mit News zu neuer Hardwae (wie dem V4 Dongle) und sonstigen Community Projekten.

Mit dem Parameter `--net` kannst du sogar `http://localhost:8080/` im Browser öffnen und siehst die Flieger live auf einer Karte.
{{< highlight bash >}}
dump1090 --interactive --net
{{< /highlight >}}

{{< figure
    src="/images/dump1090.png"
    alt="dump1090 Webinterface"
    caption="Live-Radar im Browser via dump1090"
    class="ma0 w-75"
>}}

## Grafische Tools und Fortgeschrittenes

### Kismet-Integration
Wie schon erwähnt, kann Kismet den RTL-SDR als Datenquelle für alles Mögliche nutzen, was kein WLAN ist.

* **Für IoT/Sensoren:** `kismet -c source=rtl433-0:type=rtl433`

{{< figure
    src="/images/kismet_rtl433.png"
    alt="Kismet 433 meters"
    caption="Temperatursensoren aufgespürt mit rtl433 Datasource"
    class="ma0 w-75"
>}}

* **Für Flugzeuge:** `kismet -c source=rtladsb-0:type=rtladsb`

{{< figure
    src="/images/kismet_rtladsb.png"
    alt="Kismet ADSB"
    caption="Flugzeuge aufgespürt mit ADSB datasource"
    class="ma0 w-75"
>}}

### Demnächst: SDRpp und Gqrx
Konsole ist cool, aber manchmal braucht man einfach ein ordentliches Wasserfall-Diagramm, um zu sehen, "wo" gerade was los ist.

Ich bastel gerade daran, **SDRpp** (eine moderne, richtig fixe SDR-Software) ordentlich auf OpenBSD zum Laufen zu bringen. Es ist noch "Work in Progress", sieht aber schon sehr gut aus. Auch den **Gqrx**-Port schaue ich mir gerade an. Die Updates dazu landen regelmäßig in meinem [mystuff](https://github.com/buzzdeee/mystuff/tree/main/comms) Repo auf GitHub.

### Fazit & Ausblick

Wie ist der Stand der Dinge für Funker auf OpenBSD?

* **Läuft super:** Die CLI-Tools wie `rtl-sdr`, `rtl_433` und `dump1090` sind grundsolide.
* **Kismet:** Funktioniert top für ADS-B und Wettersensoren.
* **Der "GNURadio"-Elefant:** GNURadio gibt's zwar in den Ports, aber die Abhängigkeiten sind riesig und die Performance unter OpenBSD ist wegen des komplexen Threadings manchmal etwas zickig.
* **Zukunft:** Behaltet **[SDRpp](https://www.sdrpp.org/)** im Auge. Ich arbeite hart daran, dass wir auf OpenBSD bald eine moderne, schicke GUI für den Funk-Alltag haben.

{{< figure
    src="/images/sdrpp.png"
    alt="SDR++ auf OpenBSD"
    caption="SDR++ auf OpenBSD"
    class="ma0 w-75"
>}}

OpenBSD zeigt mal wieder: Es ist eine großartige Plattform für Hardware-Basteleien, solange man bereit ist, **tief in die Materie einzutauchen** und über den einen oder anderen verlorenen Datenpunkt hinwegzusehen!
