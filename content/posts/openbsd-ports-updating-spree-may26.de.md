---
title: "OpenBSD Ports Updating Spree: OpenVox, DFIR Tools und Rocrail-Rettung"
subtitle: "Der unendliche Kampf gegen den Backlog an veralteten Packages ;)"
date: 2026-05-22T19:45:44+02:00
lastmod: 2026-05-22T19:45:44+02:00
draft: false
description: "Ein Blick auf den OpenBSD-Ports-Update-Marathon dieser Woche: Einige wichtige Updates für OpenVox, Volatility3, plaso, sleuthkit und Kismet und wie die Modelleisenbahn wieder auf die Schiene geholt wurde."
license: "MIT"
images: [ "/images/ports-updating-sprint.png" ]

tags: ["OpenBSD", "Ports", "Security", "Ruby", "Puppet", "Rocrail"]
categories: ["Tech", "Porting"]

featuredImage: "/images/ports-updating-sprint.png"
featuredImagePreview: "/images/ports-updating-sprint.png"

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
  images: [ "/images/ports-updating-sprint.png" ]
---

Ich habe diese Woche einiges an Zeit in einen gezielten OpenBSD-Ports-Update-Sprint gesteckt und dafür ein paar andere Projekte beiseitegeschoben, um diesen überfälligen Kram endlich mal wegzubügeln. Wie das meistens so läuft: Was mit ein paar einfachen Versions-Bumps anfing, endete schnell in einem fetten Domino-Effekt aus miteinander verknüpften Dependencies – besonders im Ruby- und Python-Ökosystem ;)

Es lohnt sich nicht, jeden kleinen Minor-Bump einzeln aufzuzählen, aber hier ist die Übersicht der wichtigsten Updates, die frisch im Tree gelandet sind.

## OpenVox & Puppet-Ökosystem

Rund um die `sysutils/openvox-server`-Architektur und deren Umfeld gab es diesmal ordentlich was zu tun. 

* **openvox-server** wurde auf 8.13.0 aktualisiert, zusammen mit **r10k** (5.0.3) und **puppetboard** (7.0.2).
* **ruby-pdk** und seine Dependencies haben einiges an Arbeit gekostet. Ich habe eine ganze Reihe von Abhängigkeiten hochgezogen und die Versionen entsprechend gelockert. Seit dem mittlerweile nicht mehr ganz so frischen Wechsel von Puppet zu OpenVOX sind Teile von PDK zwar kaputt, aber es taugt immer noch super, um mal eben schnell ein nacktes Modul oder Klassen aus dem Boden zu stampfen, um loszulegen.

## Netzwerk, Security & Forensik

Die Digital-Forensik- (DFIR) und Wireless-Security-Stacks haben einen fetten Refresh bekommen. 

* **[volatility3](https://github.com/volatilityfoundation/volatility3)** (2.28.0), **[pdf-parser](https://blog.didierstevens.com/my-software/)** (0.7.14), **[py-fickling](https://github.com/trailofbits/fickling)**, **[apktool](https://apktool.org/)** und **[exploitdb](https://gitlab.com/exploit-database/exploitdb)** sind alle auf dem neuesten Stand.
* **[kismet](https://www.kismetwireless.net/)**: Hier gab es ein paar coole Verbesserungen. Kismet linkt jetzt gegen `libpcap` aus den Ports, wodurch sich `kismetdb`-Captures sauber in das `pcap`-Format für BTLE- und NRF-Captures konvertieren lassen. Außerdem habe ich den [Zigbee: TICC 2531](https://www.kismetwireless.net/docs/readme/datasources/zigbee-ticc2531/)-Support ans Laufen gebracht. Damit lässt sich der [TI CC2531 Zigbee](https://www.ebay.de/itm/404481824304?var=674447321503&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=TICC2531&toolid=10001&mkevt=1#affiliate)-Stick nutzen, um Zigbee-basierte Funknetzwerke abzuscannen. Wer dazu mehr Details wissen will: Ich habe das Setup vor einiger Zeit in einem eigenen Post gecovert: [Zigbee-Sniffing unter OpenBSD: Deep Dive mit dem TI CC2531](/zigbee-sniffing-on-openbsd/).
* **[plaso](https://github.com/log2timeline/plaso)** & **[sleuthkit](https://sleuthkit.org/sleuthkit/)**: Sleuthkit ging hoch auf 4.15.0 und Plaso auf 20260512. Das zog das Update der kompletten darunterliegenden Python-Dependency-Kette nach sich (`py-acstore`, `py-dtfabric`, `py-dfvfs`, etc.). 
* Als ich so an `sysutils/py-tsk` saß, ist mir aufgefallen, dass ich das Update im Februar glatt verpennt hatte. Ich habe Nägel mit Köpfen gemacht und mich direkt offiziell als **MAINTAINER** dafür eingetragen.

## Modelleisenbahn, Audio und Spiele

Ports-Arbeit besteht ja nicht nur aus Server-Automatisierung und Security. Ab und zu braucht man (oder die Kids) auch mal eine Pause ;) Also habe ich mich auch um ein paar Entertainment-Pakete gekümmert.

* **[rocrail](https://www.rocrail.online/doku.php?id=start)**: Das war tatsächlich seit einigen Jahren kaputt. Ich habe es jetzt so weit aktualisiert, dass es wieder sauber baut und läuft, zusammen mit einer Handvoll anderer Detail-Verbesserungen unter der Haube. Über die ganze Aktion hatte ich schon mal im Artikel [Gleise und Dämonen: Digitale Modelleisenbahn unter OpenBSD](/digital-model-railroading-on-openbsd/) geschrieben.
* **[qsynth](https://qsynth.sourceforge.io/qsynth-index.html)**: Update von 1.0.3 auf 1.0.5.
* **games**: Kleinere Version-Bumps für **[emptyclip](https://empty-clip.gitlab.io/)** und **[choria](https://choria.gitlab.io/)**.

## Wo ich schon mal dabei war...

Auf dem Weg habe ich auch direkt noch ein paar andere Baustellen aufgeräumt:
* Die Java-Dependency für **devel/jenkins** wurde auf Java 21 angehoben.
* Die Abhängigkeiten von `ruby-fast_gettext` in **sysutils/ruby-openvox/8** wurden etwas gelockert.

## Fazit / Wrapping up

Nach diesem Sprint ist der Backlog an veralteten Ports erst mal drastisch geschrumpft. Auf Null ist er zwar immer noch nicht – aber seien wir mal ehrlich, das wird er wahrscheinlich nie sein ;) Die Zeit tickt unaufhaltsam weiter und wird die Liste ganz "magisch" wieder von alleine wachsen lassen...

Bis dahin ist jetzt aber erst mal alles in den Snapshots verfügbar, committed und einsatzbereit. Lasst die Updates laufen und sagt Bescheid, falls irgendwo was knallt!

