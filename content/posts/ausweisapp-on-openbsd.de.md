---
title: "AusweisApp auf OpenBSD: Wenn LibreSSL nicht will, muss eben Wine herhalten"
subtitle: "Ein Erfahrungsbericht zwischen gescheiterten Ports, MSI-Extraktionen und IPv4-zu-IPv6-UDP-Broadcast-Brücken"
date: 2026-11-16T21:05:54+01:00
lastmod: 2026-11-16T21:05:54+01:00
draft: false
description: "Wie man die deutsche AusweisApp trotz fehlender RSA-PSK-Unterstützung in LibreSSL mithilfe von 64-Bit-Wine und socat auf OpenBSD zum Laufen bekommt."
license: "MIT"
images:
  - "/images/Ausweisapp_on_OpenBSD.jpg"
  - "/images/AusweisApp_Installer.png"
  - "/images/AusweisApp_SmartphonePairing.png"
  - "/images/AusweisApp_SmartphoneSettings.png"

tags:
  - OpenBSD
  - Wine
  - AusweisApp
  - eID
  - Networking
  - socat
categories:
  - OpenBSD
  - Tutorials

featuredImage: "/images/Ausweisapp_on_OpenBSD.jpg"
featuredImagePreview: "/images/Ausweisapp_on_OpenBSD.jpg"

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
    - "/images/Ausweisapp_on_OpenBSD.jpg"
    - "/images/AusweisApp_Installer.png"
    - "/images/AusweisApp_SmartphonePairing.png"
    - "/images/AusweisApp_SmartphoneSettings.png"
  # ...
---

## Einleitung und Motivation

Jahrelang lag er da: dieser unscheinbare Brief mit dem Hinweis, dass ich die Online-Ausweisfunktion meines Personalausweises freischalten kann. Die Idee dahinter ist ja durchaus charmant – nicht mehr wegen jeder Kleinigkeit einen Termin beim Amt buchen müssen, sondern Behördengänge, Elster oder BundID bequem vom heimischen Schreibtisch aus erledigen. 

Als ich dann endlich einmal die Zeit und Muße fand, mich der Sache anzunehmen, folgte auf die anfängliche Begeisterung allerdings sehr schnell die Ernüchterung. Als OpenBSD-Nutzer ist man es gewohnt, gelegentlich etwas Hand anzulegen, wenn Software primär für Linux, Windows oder macOS entwickelt wird. Doch die **[AusweisApp](https://de.wikipedia.org/wiki/AusweisApp_(Governikus))** erwies sich als besonders hartnäckiger Kandidat.

## Der erste Versuch: Ein nativer OpenBSD-Port

Zunächst sah die Ausgangslage gar nicht so schlecht aus: Freudig stellte ich fest, dass der Quellcode der AusweisApp frei verfügbar ist und bereits fertige Pakete für diverse Linux-Distributionen existieren. Das stimmte mich vorsichtig optimistisch, dass sich die Anwendung ohne allzu große Verrenkungen auch für OpenBSD bauen lassen müsste.

Bereits im Dezember 2024 hatte ich mich daran versucht, die damalige Version **2.4.0** der AusweisApp nativ für OpenBSD zu portieren. Das Unterfangen scheiterte jedoch schon beim Bauen grandios an den kryptografischen Anforderungen:

{{< highlight text >}}
-- Could NOT find Qt6QmlCompilerPlusPrivateTools (missing: Qt6QmlCompilerPlusPrivateTools_DIR)
CMake Error at cmake/Libraries.cmake:183 (message):
  Your OpenSSL library looks incompatible:

  Change Dir:
  '/home/ports/pobj/amd64/AusweisApp-2.4.0/build-amd64/CMakeFiles/CMakeTmp'

  Run Build Command(s): /usr/local/bin/ninja -v cmTC_125bc

  [1/2] /home/ports/pobj/amd64/AusweisApp-2.4.0/bin/c++ -DQT_CORE_LIB
  -DQT_NETWORK_LIB -DQT_NO_DEBUG -isystem
  /usr/local/include/X11/qt6/QtNetwork -isystem /usr/local/include/X11/qt6
  -isystem /usr/local/include/X11/qt6/QtCore -isystem
  /usr/local/lib/qt6/mkspecs/openbsd-clang -O2 -pipe -g -O0 -fPIC -MD -MT
  CMakeFiles/cmTC_125bc.dir/openssl.cpp.o -MF
  CMakeFiles/cmTC_125bc.dir/openssl.cpp.o.d -o
  CMakeFiles/cmTC_125bc.dir/openssl.cpp.o -c
  /home/ports/pobj/amd64/AusweisApp-2.4.0/AusweisApp-2.4.0/cmake/tests/openssl.cpp

  FAILED: CMakeFiles/cmTC_125bc.dir/openssl.cpp.o
  ...
  /home/ports/pobj/amd64/AusweisApp-2.4.0/AusweisApp-2.4.0/cmake/tests/openssl.cpp:26:3:
  error: RSA-PSK is required.

     26 |         #error RSA-PSK is required.
        |          ^

  1 error generated.
  ninja: build stopped: subcommand failed.
{{< /highlight >}}

{{< admonition type="info" title="Warum ein Build gegen OpenSSL nicht hilft" >}}
OpenBSD setzt bekanntermaßen auf **LibreSSL**, welches aus guten Gründen kein `RSA-PSK` unterstützt. Man könnte nun auf die Idee kommen, die AusweisApp einfach gegen das separat verfügbare OpenSSL-Paket aus den Ports zu kompilieren. Da die AusweisApp jedoch gegen **Qt6** linkt und Qt6 unter OpenBSD wiederum gegen LibreSSL gebaut ist, führt das Mischen der beiden SSL-Bibliotheken spätestens zur Laufzeit zu unschönen Abstürzen und Symbol-Konflikten.
{{< /admonition >}}

An diesem Punkt habe ich das Projekt erst einmal frustriert beiseitegelegt.

## Neuer Anlauf: 64-Bit-Wine unter OpenBSD

Vor kurzem hat sich die Ausgangslage jedoch geändert: **Wine** wurde in den OpenBSD-Ports-Tree aufgenommen – inklusive Unterstützung für **64-Bit-Windows-Anwendungen**! Das schrie förmlich nach einem zweiten Versuch, diesmal mit der [aktuellen Windows-Version der AusweisApp](https://www.ausweisapp.bund.de/download) (Version 2.6.2).

### Die Hürde mit dem MSI-Installer

Beim direkten Start des `.msi`-Installers begrüßte mich Wine zunächst mit ein paar harmlosen Meldungen auf der Konsole, öffnete aber immerhin ein grafisches Fenster:

{{< highlight text >}}
00a4:err:ntoskrnl:ZwLoadDriver failed to create driver L"\\Registry\\Machine\\System\\CurrentControlSet\\Services\\winebth": c00000e5
003c:fixme:service:scmdatabase_autostart_services Auto-start service L"winebth" failed to start: 1359
00e4:fixme:file:NtLockFile I/O completion on lock not implemented yet
00e4:fixme:ntdll:NtQuerySystemInformation info_class SYSTEM_PERFORMANCE_INFORMATION
{{< /highlight >}}

Die Freude währte kurz: Der Installer behauptete steif und fest, ich bräuchte mindestens ein 64-Bit Windows 10 oder Windows Server 2016.

{{< figure src="/images/AusweisApp_Installer.png" alt="Fehlermeldung des AusweisApp Installers unter Wine" caption="Der MSI-Installer verweigert den Dienst wegen angeblich zu alter Windows-Version." class="ma0 w-75" >}}

Also habe ich mit `winecfg` explizit Windows 10 eingestellt und dies auch über die Registry verifiziert:

{{< highlight shell >}}
winecfg -v win10
wine reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion" /v ProductName
{{< /highlight >}}

Die Ausgabe bestätigte brav: `ProductName REG_SZ Microsoft Windows 10`. Trotzdem zeigte der Installer beim nächsten Start exakt dieselbe Fehlermeldung. 

Statt weiter mit der Installationsroutine Argumente auszutauschen, wer's wohl besser weiß, lässt sich ein MSI-Paket glücklicherweise auch einfach administrativ entpacken und die Anwendung direkt starten:

{{< highlight shell >}}
msiexec /a AusweisApp-2.6.2.msi /qb TARGETDIR="C:\AusweisApp"
wine ${HOME}/.wine/drive_c/AusweisApp/PFiles64/AusweisApp/AusweisApp.exe
{{< /highlight >}}

Und siehe da: Die AusweisApp startete ohne Murren!

### Das Netzwerk-Rätsel beim Smartphone-Pairing

Da ich keinen dedizierten USB-Kartenleser verwende, wollte ich mein iPhone (mit installierter AusweisApp) als Kartenleser über das lokale Netzwerk koppeln. Nichts einfacher als das! Denkste, im Pairing-Modus fanden sich Desktop und Smartphone einfach nicht.

Ein Blick in die Netzwerkanalyse mit `tcpdump`, `fstat -p` sowie `netstat -f inet` und `netstat -f inet6` brachte schließlich Licht ins Dunkel – und förderte ein kurioses Verhalten zutage:

* Das **iPhone** sendet seine Suchanfragen als **IPv4-Broadcast** auf **UDP-Port 24727**.
* Die **AusweisApp unter Wine** lauschte jedoch ausschließlich auf **IPv6 UDP-Port 24727** – und komplett gar nicht auf IPv4!

Selbst das explizite Deaktivieren von IPv6 auf dem Netzwerk-Interface (`ifconfig rge0 -inet6`) konnte die Anwendung nicht davon überzeugen, einen IPv4-Socket zu öffnen.

{{< figure src="/images/AusweisApp_SmartphonePairing.png" alt="Kein Smartphone zu finden weit und breit" caption="Kein Smartphone zu finden weit und breit" class="ma0 w-75" >}}


### Die Lösung: Eine Brücke zwischen IPv4 und IPv6

Wenn die App nicht auf IPv4 hören will, müssen die Pakete eben dorthin geleitet werden, wo sie zuhört. Zuerst habe ich mir dazu ein kleines Python-Skript geschrieben, das eingehende IPv4-Broadcasts auf Port 24727 entgegennimmt und an `[::1]:24727` weiterleitet.

Noch deutlich eleganter und ohne zusätzliches Skript lässt sich das Problem jedoch als Einzeiler mit **socat** lösen:

{{< highlight shell >}}
socat UDP4-RECVFROM:24727,broadcast,fork UDP6-SENDTO:[::1]:24727
{{< /highlight >}}

{{< admonition type="tip" title="Praktischer Alias für den Alltag" >}}
Damit man nicht jedes Mal zwei Terminals öffnen muss, bietet sich ein kleiner Alias in der eigenen Shell-Konfiguration (z. B. `~/.profile`) an:

`alias ausweisapp="socat UDP4-RECVFROM:24727,broadcast,fork UDP6-SENDTO:[::1]:24727 & wine ${HOME}/.wine/drive_c/AusweisApp/PFiles64/AusweisApp/AusweisApp.exe"`

*(Hinweis: Damit die Kommunikation zwischen Smartphone und Desktop klappt, muss die Firewall bzw. `pf` eingehenden Traffic auf **UDP-Port 24727** zulassen.)*
{{< /admonition >}}

Sobald die Weiterleitung lief, funktionierte die Kopplung mit dem iPhone auf Anhieb! Ich konnte meine Transport-PIN ändern, meine Ausweisdaten auslesen und mich anschließend direkt über Firefox auf OpenBSD bei **BundID**-Diensten sowie bei **ELSTER** anmelden.

{{< figure src="/images/AusweisApp_SmartphoneSettings.png" alt="Pairing dann endlich doch erfolgreich." caption="Pairing dann endlich doch erfolgreich." class="ma0 w-75" >}}

## Fazit und Ausblick

Es ist zwar ein kleiner Umweg über Wine und ein etwas sperriger Workaround mit `socat` nötig, aber unterm Strich zählt das Ergebnis: Die Online-Ausweisfunktion funktioniert jetzt zuverlässig unter OpenBSD. Ein kleiner Wermutstropfen bleibt allerdings: Der Wine-Prozess der AusweisApp genehmigt sich selbst im Leerlauf quasi permanent 100 % Auslastung auf einem CPU-Kern. Da man die Anwendung aber ohnehin nur kurz für den jeweiligen Authentifizierungsvorgang braucht und danach zeitnah wieder schließen kann, lässt sich das im Alltag gut verschmerzen.

Aktuell versuche ich noch über die `ports@`-Mailingliste herauszufinden, ob sich das Netzwerkverhalten direkt in Wine unter OpenBSD beheben lässt, oder ob es schlicht daran liegt, wie die AusweisApp selbst ihren Socket öffnet – was den Handlungsspielraum natürlich einschränken würde. 

Und vielleicht gibt es sogar noch Hoffnung auf eine ganz saubere Lösung: `kirill@` hat sich bei mir gemeldet, weil er eventuell noch einen älteren, halbfertigen Port der AusweisApp herumliegen hat, in dem er genau jenes `RSA-PSK`-Problem bereits adressiert hatte. Mal schauen, ob sich daraus langfristig doch noch ein echter nativer Port für OpenBSD ergibt. Die Zeit wird es zeigen!

