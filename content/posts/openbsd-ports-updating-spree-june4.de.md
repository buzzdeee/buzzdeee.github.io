---
title: "Zwei Wochen OpenBSD Ports maintenance: LLVM 22 Fallout, SDR Stack Expansion, und Ruby Housekeeping"
subtitle: ""
date: 2026-06-04T22:32:06+02:00
lastmod: 2026-06-04T22:32:06+02:00
draft: true
description: "A retrospective on recent OpenBSD ports maintenance: resolving LLVM 22 fallout across GNUstep, expanding Software Defined Radio (SDR) tools, introducing GNUstep App Wrappers, and streamlining Ruby web scanners."
license: "MIT"
images: ["/images/openbsd-ports-updating-spree-june4.png"]

tags: ["OpenBSD", "Ports", "GNUstep", "LLVM", "SDR", "Ruby", "Security"]
categories: ["Tech", "Porting"]

featuredImage: "/images/openbsd-ports-updating-spree-june4.png"
featuredImagePreview: "/images/openbsd-ports-updating-spree-june4.png"

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
library:
  css:
  js:
seo:
  images: [ "/images/openbsd-ports-updating-spree-june4.png" ]
---

Hinter mir liegen wieder mal zwei Wochen im OpenBSD Ports Tree. Zwischen größeren Toolchain-Updates im Base-System, die bei mir für einiges an Fallout gesorgt haben, dem Ausbau des Software Defined Radio (SDR) Ökosystems, Updates für mein Repository an GNUstep App Wrappern, einem Major-Update von wpscan sowie einem überfälligen Update von whatweb inklusive dem nötigen Dependency-Maze gab es für mich jede Menge zu tun. 

Hier ist ein Breakdown aller Updates, neuen Imports und Maintenance-Arbeiten der letzten 14 Tage.

---

## Das LLVM 22 Upgrade & der GNUstep Fallout

Das Update von **LLVM auf Version 22** im OpenBSD Base-System war ein großartiger Meilenstein, aber wie das bei großen Compiler-Sprüngen eben so ist: Es brachte eine ganze Welle strengerer Kompilier-Regeln und Deprecations mit sich. Als Maintainer eines beträchtlichen Teils unseres **GNUstep**-Ökosystems durfte ich also zusehen, wie die Build-Infrastruktur reihenweise mit Fehlern aufleuchtete. 

Das Ganze zu fixen hat einiges an Zeit in Anspruch genommen. Mein Workflow dabei sah im Grunde immer gleich aus: Zuerst die eigentliche Ursache analysieren und untersuchen, woran der Build scheitert. Danach prüfen, ob upstream bereits Fixes existieren – falls ja, habe ich diese direkt übernommen; falls nein, musste ich eine eigene Lösung finden und diese anschließend upstream einreichen.

### Fixed und Upstreamed
Für etliche Ports wurden die Patches, die ich gegen die LLVM 22 Build-Fehler geschrieben habe, bereits erfolgreich im jeweiligen Upstream gemergt, oder meine PRs warten gerade noch auf Review:
* `x11/gnustep/gworkspace` - GNUstep workspace manager
* `x11/gnustep/lapispuzzle` - tetris like puzzle game
* `x11/gnustep/netclasses` - asynchronous networking framework for GNUstep
* `x11/gnustep/examples` - GNUstep example applications

### Lokale Port-Fixes
Bei anderen Ports habe ich die Fixes entweder direkt von Upstream übernommen oder sie waren es schlicht nicht wert, extra upstream eingereicht zu werden:
* `x11/gnustep/base` — GNUstep base library
* `x11/gnustep/gui` — GNUstep gui library
* `x11/gnustep/pantomime` — framework to major mail protocols
* `x11/gnustep/paje` — GNUstep based trace visualization tool
* `x11/gnustep/gmastermind` — GNUstep mastermind game
* `x11/gnustep/renaissance` — GNUstep layer to write portable GUIs

### Rescuing Gomoku
Eine ganz besondere Challenge wartete bei `x11/gnustep/gomoku`. Als ich mich daran setzen wollte, das Ganze zu patchen, stellte ich fest, dass das ursprüngliche Upstream-Repository aus dem Internet verschwunden war. Bevor ich den Port sterben lasse, habe ich mich kurzerhand mit dem Maintainer des **GAP (The GNUstep Application Project)** kurzgeschlossen. Wir haben Gomoku direkt in das GAP-Projekt importiert und so seine langfristige Zukunft und ein neues Upstream-Zuhause gesichert.

---

## Jede Menge neue GNUstep App Wrapper im Repository

Um die Fixes im Kern-Ökosystem abzurunden und dafür zu sorgen, dass sich auch Nicht-GNUstep Desktop-Anwendungen nahtlos in eine saubere Workstation im GNUstep Desktop einfügen, habe ich meine Sammlung von **GNUstep App Wrappers** erweitert. 

Diese Wrapper erlauben es, ganz normale grafische Apps nativ in `GWorkspace` oder mein GNUstep-Dock zu integrieren. Meine komplette Collection ist auf [GitHub](https://github.com/buzzdeee/gnustep_apps) gehostet.

Neu hinzugekommen sind Wrapper-Konfigurationen für folgende Programme:
* `audacity` — Audio editing suite
* `cubicsdr` — Spectrum visualizer (passt direkt zu den neuen SDR-Ports weiter unten!)
* `ghidra` — NSA's reverse engineering framework
* `gimp` — GNU Image Manipulation Program
* `luanti` — The voxel game engine (ehemals bekannt als Minetest)
* `prusaslicer` — 3D printing slicer suite
* `rocview` — Rocrail train control system UI
* `spdrs60` — SpdrS60 railway interlocking simulator
* `xclicker` — X11 fast auto-clicker
* `xtrkcad` — CAD program for designing model railroad layouts

---

## Expanding the Software Defined Radio (SDR) Ecosystem

Das Thema Software Defined Radio auf OpenBSD hat einen massiven Push bekommen. Mein primäres Ziel dieser Aktion war es, mächtige SDR-Anwendungen wie **[CubicSDR](https://github.com/cjcliffe/CubicSDR)** und **[Gqrx](https://www.gqrx.dk/)** samt der dafür notwendigen Runtime-Infrastruktur komplett zu portieren. 

Das Herzstück dieses Ökosystems ist **[SoapySDR](https://github.com/pothosware/SoapySDR/wiki)**, das als herstellerunabhängige Hardware-Abstraktionsschicht dient. Indem ich SoapySDR zusammen mit den einzelnen Hardware-Modulen portiert habe, bietet OpenBSD nun eine einheitliche API. Jede App, die gegen SoapySDR geschrieben wurde, kann sauber mit den unterschiedlichsten Funkgeräten kommunizieren – ganz ohne spezielle, individuelle Treiber. Vorerst gilt das zumindest erstmal für **[RTL-SDR](https://www.rtl-sdr.com/)**- sowie **[HackRF One](https://greatscottgadgets.com/hackrf/one/)**- und **[HackRF Pro](https://greatscottgadgets.com/hackrf/pro/)**-Geräte.

### Neue Imports
* `comms/cubicsdr`: A beautiful cross-platform Software-Defined Radio application built using wxWidgets.
* `comms/soapysdr`: The vendor-neutral SDR data abstraction library.
* `comms/soapy-rtlsdr`: Wrapper to bring RTL-SDR dongle support to SoapySDR.
* `comms/soapy-hackrf`: Wrapper enabling HackRF hardware under the Soapy framework.

### Updates und neue Zuständigkeiten
* `comms/hackrf`: Aktualisiert, um neueste Firmware und Runtime-Anforderungen zu unterstützen, allem voran mit Support für das **HackRF Pro**.
* `comms/dump1090`: Update für den beliebten ADS-B Flugzeugtracker. Weil ich gerade eh schon unter der Haube zugange war, habe ich gleich Nägel mit Köpfen gemacht und mich als offizieller **MAINTAINER eingetragen**, um den Port auch in Zukunft gut zu versorgen.

### Awaiting Review auf `ports@`
Um meinen SDR-Rundumschlag komplett zu machen, habe ich noch drei wichtige Frontend-Erweiterungen auf die Mailingliste geworfen:
* `comms/gqrx`: A powerful AM/FM/SSB software-defined radio receiver graphical user interface.
* `comms/gr-osmosdr`: The GNU Radio block for various radio hardware.
* `comms/sdrpp`: bloat-free SDR software with a focus on performance

Wenn du ein SDR-Setup unter OpenBSD betreibst, mit HackRF oder RTL-SDR: Schau auf ports@ und teste, Feedback welcome ;)

Passend zum Thema SDR sei auch auf meinen früheren Artikel verwiesen: [Spaß mit RTL-SDR auf OpenBSD: Von Flugzeugen, Funk-Thermometern und Radio](/fun-with-rtl-sdr/)

{{< admonition type=note title="Hinweis zu Hardware-Alternativen" open=true >}}
Für wen die HackRF-Geräte zu teuer sind: Die bekannten **RTL-SDR Blog V3/V4** Dongles gibt es auch auf eBay zu einem guten Preis. Passende Antennen findest du dort ebenfalls... Nutze dafür einfach diesen [Partner-Link zu eBay](https://www.ebay.de/sch/i.html?_nkw=RTL-SDR+Blog&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=RTLSDR&toolid=10001&mkevt=1).
{{< /admonition >}}

---

## Webscanner & Ruby-Dependency-Frühjahrsputz

Das Aktualisieren einer komplexen Ruby-Applikation gleicht oft einem Domino-Spiel. Mein Einpflegen eines Major-Updates für **wpscan** und das längst überfällige Update von **whatweb** zog mich tief in ein ordentliches Dependency-Rabbit-Hole. Aber am Ende gab es Licht und alles war aufgeräumt: Altes wurde gelöscht und der Rest auf den neuesten Stand gebracht.

### WPScan Revamp & Kahlschlag
Das Major-Update von `security/wpscan` erforderte einen Wechsel bei der Interaktion mit Headless-Browsern. Das gab mir die perfekte Gelegenheit, ein Alt-Framework über Bord zu werfen und die Dependency-Kette deutlich zu verfeinern:

* `www/ruby-ferrum`: **NEUER** Port, um High-Level Chrome/Chromium-Automatisierung bereitzustellen.
* `security/ruby-cms_scanner`: **GELÖSCHT** (wurde nur noch von der alten wpscan-Version genutzt).
* `devel/ruby-opt_parse_validator`: **GELÖSCHT** (verwaist nach dem Rauswurf von `ruby-cms_scanner`).

### Kaskadierende Ruby-Dependency-Updates
Um die Anforderungen der neuen wpscan Version zu erfüllen, musste, bzw. konnte ich dann endlich auch mal, diverse andere Ruby-Bibliotheken updaten:
* `devel/ruby-activesupport` — Updated
* `net/ruby-public_suffix` — Updated
* `net/ruby-connection_pool` — Updated
* `www/ruby-addressable` — Updated
* `www/ruby-xmlrpc` — Updated

### Übernahme von WhatWeb
* `net/whatweb`: Den Webscanner habe dabei auch auf die neueste Version gehievt. Während ich mich durch die Updates gewühlt habe, habe ich auch hier direkt den **MAINTAINER übernommen**, um den Port auf OpenBSD künftig auf der Überholspur zu halten.

---

## Diverse Updates & ein bisschen Spaß für die Kids

Um meine letzten zwei Wochen rundzumachen, bekamen noch ein paar unabhängige Komponenten im Tree etwas Liebe von mir ab:

### Infrastructure- & Anwendungs-Bumps
* `net/ruby-msgpack` & `www/ruby-faraday-net_http`: Updated (beides kritische Dependencies für `openvox`).
* `textproc/py-commonmark`: Updated (Dependency für `puppetboard`).
* `archivers/ruby-rubyzip`: Auf die neueste stabile Version angehoben.
* `audio/qsynth`: Update für das fluidsynth GUI-Frontend.
* `security/exploitdb`: Updated, damit die lokalen Exploit-Payload-Definitionen wieder frisch mit den öffentlichen Archiven synchron sind.

### Just For Fun: XClicker
Zu guter Letzt habe ich noch einen neuen Port zur Überprüfung an `ports@` geschickt:
* `x11/xclicker`: Ein schneller, feature-reicher grafischer Autoclicker für X11.

*Wozu?* Nun ja, die Kids von heute spielen diese albernen Clicker-Games am Rechner und jammern ständig, dass ihnen die Finger wehtun. Ab jetzt können sie effizient und vollkommen nativ unter OpenBSD schummeln. 😉

---

## Wrapping Up

Zwei Wochen, dutzende gebaute Pakete, Compiler-Zickereien elegant umschifft, ein robuster neuer SDR-Stack und maßgeschneiderte App-Wrapper für einen runderen Desktop-Workflow. 

Wenn du auf `-snapshots` unterwegs bist, mach einfach ein Update und probier die neuen SDR-Sachen aus. Falls du einen GNUstep-basierten Desktop nutzt, schau dir mein App-Wrapper-Repository an, um dein Setup zu vervollständigen. Und wie immer gilt: Feedback und Testergebnisse für die Ports, die auf `ports@` auf ihr Review warten, sind extrem gerne gesehen. 

*Happy porting!*

