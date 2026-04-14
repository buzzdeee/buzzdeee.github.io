---
title: "Die Lücke schließen: Modernes Kismet für OpenBSD"
subtitle: "Vom betagten ncurses-Port zum modernen Multi-Protokoll-RF-Sensor-Hub."
date: 2026-04-13T21:39:43+02:00
lastmod: 2026-04-13T21:39:43+02:00
draft: false
author: ""
authorLink: ""
description: "Schlanker Code statt #ifdef-Dschungel: Wie das moderne Kismet nach einer langen Odyssee endlich im OpenBSD-Ports-Tree gelandet ist."
license: "MIT"
images: [ "/images/modern-kismet.png", "/images/wifi-rf-equipment.jpeg" ]

tags: ["OpenBSD", "Kismet", "SDR", "Wireless", "C++", "Ports"]
categories: ["Tech", "Networking"] 

featuredImage: "/images/modern-kismet.png"
featuredImagePreview: "/images/modern-kismet.png"

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

seo:
  images: [ "/images/modern-kismet.png", "/images/wifi-rf-equipment.jpeg" ]

---

Jahrelang steckte OpenBSD in einer technischen Zeitkapsel fest. Während der Rest der Welt auf Kismet "Newcore" umstieg – mit seinem leistungsstarken webbasierten UI, verteiltem Capture und Multi-Protokoll-Unterstützung – blieb der OpenBSD-Port in der veralteten ncurses-Version stecken.

Die Core-Software ließ sich zwar, mit kleineren patches, problemlos kompilieren, aber es klaffte eine riesige Lücke: **es gab keinen Wifi-Capture-Treiber für OpenBSD.** Um voranzukommen und die antike Version endlich in den Ruhestand zu schicken, musste ich das Herzstück bauen: `capture_openbsd_wifi`.

## Navigation durch das `#ifdef`-Labyrinth

Als ich anfing, den Linux-Capture-Code als Referenz zu nutzen, stieß ich nicht nur auf komplexe Logik, sondern auf ein Minenfeld für den Präprozessor ;). Um die unzähligen Wege zu unterstützen, wie Linux Netzwerkschnittstellen verwaltet, ist der Code ein dichter Dschungel aus:

* `#ifdef HAVE_LIBNM` (Network Manager)
* `#ifdef HAVE_LINUX_WIRELESS` (Legacy Wireless Tools)
* `#ifdef HAVE_LINUX_NETLINK` (Modern Netlink)
* `#ifdef HAVE_LINUX_IWFREQFLAG`

Diese Komplexität zu warten, ist wahrscheinlich ein Vollzeitjob ;). Im krassen Gegensatz dazu war die OpenBSD-Philosophie („one way to do things“) ein echter Lichtblick.

### Der `cloc`-Vergleich: Linux vs. OpenBSD

Der Kontrast zwischen dem technischen Ballast unter Linux und der sauberen Implementierung für OpenBSD ist beeindruckend. Ein direkter Vergleich mit `cloc` (Count Lines of Code) unterstreicht die enorme Effizienz des OpenBSD-Stacks:

| Treiber-Komponente | Dateien | Zeilen Code |
| :--- | :---: | :---: |
| **Linux Wifi Capture** | 10 | **5.479** |
| **OpenBSD Wifi Capture** | 1 | **1.075** |

Dank des einheitlichen Netzwerk-Stacks von OpenBSD konnte ich den Funktionsumfang der alten Version mit **etwa 80 % weniger Code** als der Linux WiFi Capture Treiber abbilden. Statt komplexer `#ifdef` - Strukturen genügt hier sauberer, funktionaler Code, der direkt die Wireless-Schnittstelle des Kernels anspricht, herrlich!

---

## Exkurs in andere Hardware: BTLE und SDR

Sobald der Wi-Fi-Treiber stabil lief, konnte ich nicht anders, als mir auch andere potenzielle [Datenquellen](https://www.kismetwireless.net/docs/readme/datasources/datasources/) anzuschauen. Ich wollte einfach mal ausprobieren, was im Bereich RF-Monitoring unter OpenBSD alles machbar ist. Konkret schaute ich mir den **[RTL-SDR Blog V3](https://www.rtl-sdr.com/)** (damals aktuell) und den **[Adafruit Bluefruit LE Friend](https://www.adafruit.com/product/2267)** (mit der Nordic Sniffer Firmware) an. Um mit meinen Experimenten zu starten, besorgte ich mir sowohl einen **[RTL-SDR](https://www.ebay.de/sch/i.html?_nkw=RTL+SDR+BLOG&_sacat=0&_from=R40&_trksid=p2334524.m570.l1313&LH_TitleDesc=0&_odkw=RTL+SDR&_osacat=0&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=RTLSDR&toolid=10001&mkevt=1)** als auch einen **[Adafruit Bluefruit](https://www.ebay.de/sch/i.html?_nkw=Adafruit+nrf51822+Bluefruit&_sacat=0&_from=R40&_trksid=m570.l1313&LH_TitleDesc=0&_odkw=Adafruit+BTLE+Bluefruit&_osacat=0&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Bluefruit&toolid=10001&mkevt=1)** bei eBay.

Das Ganze hat auch technisch erstaunlich gut funktioniert:

* **BTLE ohne BT-Stack:** OpenBSD hat bekanntlich keinen Bluetooth-Stack. Da der `nrf51822`-Treiber jedoch direkt über Serial/USB mit der Hardware spricht, kann Kismet den BTLE-Traffic perfekt mitschneiden. Ein paar kleinere Patches waren nötig, um den `nrf51822`-Treiber unter OpenBSD stabil zum Laufen zu bringen. Damit sind jetzt Paketanalysen auf einer Plattform möglich, die von Haus aus eigentlich gar kein Bluetooth unterstützt.
* **Den RTL-SDR-Port aktualisieren**: Eigentlich wollte ich nur schauen, ob **RTL-ADSB** zum Flugzeuge-Tracken funktioniert. Da ich eh gerade dabei war, habe ich den OpenBSD-Port `comms/rtl-sdr` direkt auf Version **2.0.1** aktualisiert. Mit dem frischen Port ließen sich die Flieger dann auch problemlos in Echtzeit auf der Karte verfolgen.

## Der Weg nach Upstream

Als der Code stabil war und der `capture_openbsd_wifi` - Treiber zuverlässig lief, wollte ich das Ganze nicht nur auf meinem Rechner verstauben lassen. Ich habe ein paar Merge Requests eingereicht, die auch zügig angenommen wurden. Mir war wichtig, dass die OpenBSD-Unterstützung offiziell im Haupt-Repo landet, damit ich später keine Patches im OpenBSD-Port selbst pflegen muss. Das Fundament stand also... und dann hieß es erst mal warten. ;)

## Der Endspurt nach 2025

Das Warten auf ein offizielles Kismet-Release dauerte fast zwei Jahre. Als die Version vom **September 2025** endlich erschien, erwartete ich ein einfaches Port-Update. Weit gefehlt. In diesen zwei Jahren hatte sich die interne Architektur von Kismet so stark verändert, dass die OpenBSD-Teile nicht mehr funktionierten.

Nach einigen intensiven Debugging-Sessions fand ich heraus, was nötig war, um den Treiber wieder flott zu machen:

* **Anpassung an modernes Kismet:** Ich musste `capture_openbsd_wifi` noch einmal anfassen und ein wenig umbauen, damit der Treiber die aktuelle Art der Kommunikation mit dem Kismet-Server versteht.
* **OpenBSD als strenger Lehrer:** Während der Tests stieß ich auf diverse Fehler und Abstürze, die so nur unter OpenBSD auftraten. Das ist eben die „unbequeme“ Seite des Systems: Wenn es um Speichersicherheit und strikte Code-Ausführung geht, verzeiht OpenBSD nichts und deckt Schlampereien im Code sofort auf.
* **Neugier und `rtl_433`:** Bei meinem ersten Versuch hatte ich den `rtl_433`-Capture-Treiber völlig ignoriert. Aber dieses Mal siegte die Neugier. Sie trieb mich dazu, `rtl_433` nach OpenBSD zu portieren und die nötigen Patches auszuarbeiten, um auch diese Datenquelle voll funktionsfähig zu machen.

## Standort-Tracking

Kein Wireless-Survey ist ohne Standortdaten komplett. Ich habe ein altes **Garmin GPSMap 60CSx** als GPS-Gerät getestet, das ich schon Anfang der 2000er für Geocaching-Abenteuer genutzt hatte. Trotz des Alters der Hardware funktionierte es perfekt mit dem neuen Setup und lieferte zuverlässige Koordinaten für die erfassten Pakete. Im Vergleich zu damals sind diese Geräte heute recht günstig auf [eBay](https://www.ebay.de/sch/i.html?_nkw=Garmin+GPSMap+60CSx&_sacat=0&_from=R40&_trksid=m570.l1313&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Garmin60CSx&toolid=10001&mkevt=1) zu finden. Falls ihr euch eins zulegt, achtet darauf, dass das spezielle serielle Kabel dabei ist. Die Dinger sind klasse für Wardriving, da man auch eine externe Antenne anschließen kann.

{{< figure
    src="/images/wifi-rf-equipment.jpeg"
    alt="Wifi und RF Ausrüstung"
    caption="WiFi-Dongle, RTL-SDR v3, Adafruit Bluefruit nRF51822 Sniffer, Garmin GPSMap"
    class="ma0 w-75"
>}}

## Fazit

Die Lücke ist damit endlich zu. OpenBSD-Nutzer können das alte ncurses-Interface hinter sich lassen und auf ein modernes Wireless-IDS umsteigen. Sicher gibt es noch weitere Datenquellen, die man sich anschauen könnte – und die vielleicht noch den einen oder anderen Kniff brauchen, um sauber auf dem BSD-Stack zu laufen –, aber das Fundament steht.

Für alle, die an den Details der Reise interessiert sind – die Upstream-Patches finden sich hier: [github.com/kismetwireless/kismet (buzzdeee)](https://github.com/kismetwireless/kismet/commits?author=buzzdeee).

Haltet die Augen offen für weitere Blogposts, in denen ich Details zum Sniffing verschiedener RF-Arten unter OpenBSD zeigen werde.

**Viel Spaß beim Sniffen. Das Ganze wird im kommenden OpenBSD 7.9 Release enthalten sein.**

