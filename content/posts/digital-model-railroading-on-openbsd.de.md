---
title: "Gleise und Dämonen: Digitale Modelleisenbahn unter OpenBSD"
subtitle: "Die Wiederbelebung der DCC-Hardware und der Kampf gegen Bitrot im Ports-Tree"
date: 2026-05-02T17:19:11+02:00
lastmod: 2026-05-02T17:19:11+02:00
draft: false
description: "Eine Reise von den analogen Zügen der Kindheit bis hin zu einer vollständig digitalisierten Anlage unter OpenBSD."
license: "MIT"
images: [ "/images/model_railroad_openbsd.png", "/images/rocview_main.png", "/images/Rocrail_OpenDCC_Z1_options.png", "/images/spdrs60.png", "/images/dtcltiny.png", "/images/xtrkcad.png" ]

tags: ["OpenBSD", "ModelRailroad", "DCC", "Rocrail", "srcpd", "XTrackCAD", "C++", "Electronics"]
categories: ["Tech", "Hacking"]

featuredImage: "/images/model_railroad_openbsd.png"
featuredImagePreview: "/images/model_railroad_openbsd.png"

# Hugo LoveIt spezifische Einstellungen beibehalten
lightgallery: true
ruby: true
fraction: true
fontawesome: true
linkToMarkdown: true

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
seo:
  images: [ "/images/model_railroad_openbsd.png", "/images/rocview_main.png", "/images/Rocrail_OpenDCC_Z1_options.png", "/images/spdrs60.png", "/images/dtcltiny.png", "/images/xtrkcad.png" ]
---

## Die Welt im Miniaturformat

Wie wahrscheinlich viele Kinder meiner Generation habe ich unzählige Stunden in der Welt der analogen Modelleisenbahn verbracht. Doch für mich ging es nicht nur darum, einer Lokomotive dabei zuzusehen, wie sie im Kreis fährt. Die wahre Magie lag im **Erschaffen**: das Zusammenbauen von Plastikhaus-Bausätzen, das akribische Gestalten des Geländes und die komplexe Arbeit "unter der Platte" – das Verkabeln von Lichtern und elektromagnetischen Weichen. Es war meine erste Begegnung mit Ingenieurwesen und Projektmanagement, auch wenn mir das damals nicht bewusst war.

Irgendwann wurde die Anlage abgebaut und in Kisten verstaut. Dort blieb sie jahrelang liegen, während sich mein Fokus von physischen Schaltkreisen auf digitale verlagerte – genauer gesagt auf die Welt von Unix und OpenBSD.

## Das Revival 2009: Die DIY-Zentrale

Um 2009/10 holte ich diese Kisten schließlich wieder hervor. Beim Anblick der alten analogen Trafos wusste ich: Ich wollte modernisieren. Ich wollte **[DCC (Digital Command Control)](https://de.wikipedia.org/wiki/Digital_Command_Control)**, aber ich wollte den "Hands-on"-Geist behalten, den ich als Kind hatte.

Die Welt von DCC ist faszinierend, wird aber meist von proprietärer "Blackbox"-Hardware dominiert. Als OpenBSD-Nutzer fühlte ich mich natürlich zu etwas Offenerem hingezogen. Zu dieser Zeit entdeckte ich die Zentrale **[OpenDCC Z1](https://www.opendcc.de/elektronik/opendcc/opendcc.html)**.

Das Beste daran? Man konnte die Platine kaufen und die Komponenten selbst auflöten. Es ist unglaublich befriedigend, seine eigene Steuerzentrale von Grund auf selbst zu bauen. Ich verbrachte Wochen über der Platine, bewaffnet mit einem feinen Lötkolben und einer Lupe. Es ist eine nervenaufreibende Genugtuung, wochenlang zu löten, ohne zu wissen, ob das Board am Ende tatsächlich funktioniert oder ob man versehentlich einen kritischen Mikrocontroller mit einer Lötzinnbrücke gegrillt hat. Als die erste LED endlich blinkte und das Board auf einen Ping reagierte, war die Erleichterung riesig.

{{< figure
    src="/images/OpenDCC_Z1.jpeg"
    alt="OpenDCC Z1"
    caption="OpenDCC Z1"
    class="ma0 w-75"
>}}

Hardware ist jedoch nur die halbe Miete. Ich brauchte einen Weg, um von meinem Lieblingsbetriebssystem aus mit der Z1 zu kommunizieren.

Die Hardware war bereit, aber der "OpenBSD-Faktor" fehlte noch. 2010 war die Landschaft für Modelleisenbahnen unter OpenBSD eher eine Wüste. Um eine Anlage unter OpenBSD zu betreiben, benötigte ich einen Software-Stack. Also machte ich mich daran, die wichtigsten Tools für den OpenBSD-Ports-Tree zu portieren und zu paketieren:

* **[srcpd](https://sourceforge.net/projects/srcpd/)**: Der Daemon, der als SRCP-Protokoll-Gateway fungiert.
* **[spdrs60](https://sourceforge.net/projects/spdrs60/) & [dtcltiny](https://sourceforge.net/projects/dtcltiny/)**: Traditionelle grafische Stellpulte und Zugsteuerung.
* **[XtrkCad](https://sourceforge.net/projects/xtrkcad-fork/)**: Für die präzise Planung von Gleisplänen und Anlagen-Layouts.
* **[Rocrail](https://www.rocrail.online/)**: Die umfassende Automatisierungssuite.

Ich bekam alles zum Laufen, die Ports wurden akzeptiert und ich war bereit, die Anlage meiner Träume zu bauen. Doch dann... kam das Leben dazwischen. Kinder kamen, die Freizeit schrumpfte und die Eisenbahn landete erneut für mehr als ein Jahrzehnt in Kisten.

## Die Auferstehung (und der Bitrot)

Ende letzten Jahres fragten meine Kinder: *"Hey Papa, was ist eigentlich in den großen Kisten da in der Ecke?"* Es war soweit.

Wir holten alles heraus und bauten eine temporäre Anlage auf. Aber zehn Jahre sind in der Softwareentwicklung eine Ewigkeit. Als ich versuchte, meine alten OpenBSD-Ports zu aktualisieren, wurde ich schmerzhaft daran erinnert, wie schnell Software altert – ich wurde voll vom **Bitrot** eingeholt.

### Die Rocrail-Herausforderung

Ich musste feststellen, dass **Rocrail** vor Jahren auf ein Closed-Source-Modell umgestellt wurde. Für einen OpenBSD-Nutzer ist das normalerweise eine Sackgasse. Zudem war die Version in unserem Ports-Tree schon vor einiger Zeit als "broken" markiert worden; sie ließ sich einfach nicht mehr kompilieren, da sich die `wxWidgets`-Versionen weiterentwickelt hatten und der alte Code mit inkompatiblen Aufrufen zurückblieb.

Aber ich erinnerte mich daran, wie gut Rocrail früher funktionierte, und wollte es nicht einfach aufgeben. Ich fand einen Fork auf GitHub, der den letzten Open-Source-Zustand des Projekts bewahrt hatte, und kniete mich hinein. Ich verbrachte mehrere späte Nächte damit, den C++ Code zu hacken, `wxWidgets`-Inkompatibilitäten zu beseitigen, Compiler-Warnungen auszumerzen und das Build-System zu bändigen, bis es endlich wieder unter dem aktuellen OpenBSD funktionierte. Es ist ein ganz eigenes Triumphgefühl, nach einem Jahrzehnt der Vernachlässigung ein `clean build` zu sehen. Wer neugierig ist, kann sich die jüngsten Commits in meinem [Rocrail GitHub](https://github.com/buzzdeee/rocrail)-Repo ansehen.

## Aktueller Status: Grüne Signale und kleine Macken

Nach der "Großen Instandsetzung" habe ich den gesamten Stack erneut getestet: `srcpd`, `spdrs60`, `dtcltiny` und `Rocrail`. 

**Das Positive:**
* **XtrkCad:** Das ich über die Jahre immer aktuell gehalten habe, funktioniert weiterhin tadellos. Es bleibt ein grundsolides Werkzeug für die Gleisplanung.

{{< figure
    src="/images/xtrkcad.png"
    alt="XTrackCad unter OpenBSD"
    caption="XTrackCad unter OpenBSD"
    class="ma0 w-75"
>}}

* **Rocrail:** Es ist von den Toten auferstanden! Es läuft stabil. Interessanterweise kann ich den internen `srcpd`-Dienst in Rocrail aktivieren und die älteren Tools wie `spdrs60` und `dtcltiny` damit verbinden. Das ist ein sehr flexibles Setup.

{{< figure
    src="/images/rocview_main.png"
    alt="Rocrails Rocview unter OpenBSD"
    caption="Rocrails Rocview unter OpenBSD"
    class="ma0 w-75"
>}}

{{< figure
    src="/images/Rocrail_OpenDCC_Z1_options.png"
    alt="OpenDCC Z1 Setup in Rocrail"
    caption="OpenDCC Z1 Setup in Rocrail"
    class="ma0 w-75"
>}}

* **spdrs60 & dtcltiny:** Meine Tests zeigen, dass diese Tools in Verbindung mit dem Rocrail-`srcp`-Dienst angemessen stabil laufen.

{{< figure
    src="/images/spdrs60.png"
    alt="Spdrs60 unter OpenBSD"
    caption="Spdrs60 unter OpenBSD"
    class="ma0 w-75"
>}}

{{< figure
    src="/images/dtcltiny.png"
    alt="Dtcltiny unter OpenBSD"
    caption="Dtcltiny unter OpenBSD"
    class="ma0 w-75"
>}}

**Das weniger Positive:**
* **srcpd** scheint eine gewisse Eigenwilligkeit entwickelt zu haben. Es hat gelegentlich Probleme, eine stabile Verbindung zu meiner OpenDCC Z1 aufrechtzuerhalten und hinterlässt die Hardware manchmal in einem blockierten Zustand, der einen Power-Cycle erfordert. Da wartet wohl noch eine ausgiebige Debugging-Session im Code der seriellen Kommunikation auf mich, um diesen Fehler auszumerzen.

{{< admonition type=info title="Was ist srcpd?" open=true >}}
Der **srcpd** ([Simple Railroad Command Protocol](https://srcpd.sourceforge.net/srcp/) daemon) fungiert als universelle Schnittstelle zwischen Software und Hardware. Er abstrahiert die Komplexität der DCC-Protokolle und stellt einen Netzwerk-Server bereit.

**So verbinden sich die Clients:**
* **Protokoll:** Die Kommunikation erfolgt über ein einfaches, textbasiertes TCP-Protokoll (standardmäßig auf Port **4303**).
* **Flexibilität:** Da das Protokoll offen und dokumentiert ist, können verschiedene Clients gleichzeitig verbinden. Während **Rocrail** die Automatisierung übernimmt, können Tools wie **spdrs60** (als virtuelles Stellpult) oder **dtcltiny** (als Handregler) parallel auf dieselbe Hardware zugreifen.
* **Modularität:** Unter OpenBSD erlaubt dies ein sehr modulares Setup, bei dem der Daemon die serielle Schnittstelle zur Z1 exklusiv verwaltet, während die GUI-Programme als schlanke Netzwerk-Clients agieren.
{{< /admonition >}}

## Wie geht es weiter?

Trotz der kleinen Macken ist es unglaublich belohnend zu sehen, wie sich eine Lokomotive über die Gleise bewegt – gesteuert durch einen Befehl von einer OpenBSD-Maschine mit Code, den ich selbst mitgepflegt habe.

Die Mission ist noch nicht zu Ende. Während die Reaktivierung von Rocrail es nicht mehr rechtzeitig in das kommende OpenBSD 7.9 Release geschafft hat, habe ich erst heute eine E-Mail an `ports@` geschickt, um den aktualisierten Port reviewen zu lassen. Meine Hoffnung ist, ihn nach dem Release-Unlock offiziell wieder in den Tree aufzunehmen.

Ich habe mir außerdem gerade einige **[S88-Rückmeldemodule](https://shop.dcc-versand.de/bausatz-as88-gleisbesetztmelder-stromsensor-p-1872.html)** gekauft, um sie an die Z1 anzuschließen. S88 ist definitiv "Old-School"-Technik, aber sie wird von der Z1 und dieser Version von Rocrail hervorragend unterstützt. Damit ist eine einfache Gleisbelegtmeldung möglich – man weiß also, ob ein Gleisabschnitt besetzt ist.

Obwohl die Software anscheinend eine gewisse Unterstützung für **BiDi (Bi-Directional Communication)** bietet – was es dem System ermöglichen würde, genau zu identifizieren, *welcher* Zug sich in einem Block befindet – erfordert das entsprechende Decoder (die ich noch nicht habe!). Fürs Erste ist das Ziel simpel: die klassische S88-Rückmeldung an der Z1 zuverlässig zum Laufen zu bringen.

Abseits von Rocrail werfe ich auch einen Blick auf **[JMRI](https://www.jmri.org/)**. Es ist ein mächtiges Werkzeug, besonders für die Decoder-Programmierung. Allerdings bin ich noch etwas skeptisch, was den Zugriff auf die serielle Schnittstelle unter Java auf OpenBSD angeht – ein Problem, über das ich erst kürzlich stolperte, als ich mir **[openHAB](https://www.openhab.org/)** ansah. Dort verweigerte `rxtx` komplett den Dienst. Ob `purejavacomm` hier die Rettung ist, wird sich zeigen müssen.

Genau wie damals als Kind ist das Projekt nie wirklich "fertig" – und genau das ist der Punkt. Rechnet also bald mit weiteren Hacking-Updates zur Modelleisenbahn!

---
*Fragen zu DCC unter BSD? Meldet euch auf den üblichen Kanälen!*
