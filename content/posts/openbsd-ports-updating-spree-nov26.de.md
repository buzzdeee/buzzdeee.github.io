---
title: "OpenBSD Ports-Sprint: Von OpenVox-Security-Fixes bis Kismet & Plaso"
subtitle: "Ein weiterer Rundumschlag im OpenBSD Ports-Tree mit kritischen Sicherheitsupdates, Ruby-Gems und Forensik-Tools"
date: 2026-11-23T21:10:28+01:00
lastmod: 2026-11-23T21:10:28+01:00
draft: false
description: "Zusammenfassung eines umfangreichen OpenBSD Ports-Update-Sprints rund um OpenVox, WPScan, Plaso, Kismet, GNUstep und openHAB."
license: "MIT"
images:
  - "/images/openbsd-ports-update-spree-nov26.jpg"
  - "/images/puppetboard-in-action.png"
  - "/images/kismet_in_action.png"

tags:
  - OpenBSD
  - Ports
  - OpenVox
  - Security
  - WPScan
  - Plaso
  - Kismet
  - Ruby
  - SDR
categories:
  - OpenBSD
  - Porting

featuredImage: "/images/openbsd-ports-update-spree-nov26.jpg"
featuredImagePreview: "/images/openbsd-ports-update-spree-nov26.jpg"

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
    - "/images/openbsd-ports-update-spree-nov26.jpg"
    - "/images/puppetboard-in-action.png"
    - "/images/kismet_in_action.png"
---

## Es war mal wieder Zeit …

... für einen ausgiebigen OpenBSD Ports-Update-Sprint! Ein regelmäßiger Blick auf meine [Portroach-Übersicht](https://portroach.openbsd.org/sebastian%20reitenbach%20%3Csebastia@openbsd.org%3E.html) erinnert mich zuverlässig daran, wann die Liste der veralteten Pakete wieder etwas zu lang geworden ist und es höchste Zeit für den nächsten Durchgang wird. Wie so oft beginnt man eigentlich ganz harmlos mit ein paar einzelnen Paket-Updates – und ehe man sich versieht, zieht man einen ganzen Rattenschwanz an Abhängigkeiten, Security-Advisories und Upstream-Patches hinter sich her. Genau diese Mischung aus Detektivarbeit, Dependency-Jonglage und dem guten Gefühl, den Ports-Tree aktuell und sicher zu halten, macht den Reiz aus.

In diesem Durchgang lag der Fokus stark auf dem **OpenVox**-Ökosystem (dem Community-Fork von Puppet), dicht gefolgt von einer Reihe spannender **Security- und Forensik-Tools** wie Plaso, Kismet und WPScan sowie einigen Klassikern aus der GNUstep- und Smart-Home-Ecke.

---

## Das OpenVox- und Puppet-Ökosystem

**[OpenVox](https://docs.openvoxproject.org/openvox/8.x/)** ist der Open-Source-Community-Fork von Puppet und dient zur deklarativen Konfigurationsverwaltung und Automatisierung von Server-Infrastrukturen. Zusammen mit **OpenVoxDB** (zur Speicherung von Inventar- und Report-Daten) und **Puppetboard** (dem Web-Frontend zur Visualisierung) bildet es das Rückgrat vieler automatisierter Setups.

### Kritische Sicherheitsupdates & Kettenreaktionen

Vor einigen Wochen wies mich Klemens (`kn@`) auf kritische Sicherheitslücken in `openvoxdb` und `openvox-server` hin. Verantwortlich dafür war eine Schwachstelle in `trapperkeeper-authorization` ([GHSA-q4hm-33rc-572g](https://github.com/OpenVoxProject/trapperkeeper-authorization/security/advisories/GHSA-q4hm-33rc-572g)) mit einem stolzen **CVSS 3.1 Score von 9.5**.

{{< admonition type=danger title="Kritische Sicherheitslücke (CVSS 9.5 & 7.7)" >}}
Aufgrund der Schwachstelle in `trapperkeeper-authorization` (CVSS 9.5) sowie einer kurz darauf entdeckten weiteren Lücke direkt in OpenVoxDB ([GHSA-859m-7qpm-85p7](https://github.com/OpenVoxProject/openvoxdb/security/advisories/GHSA-859m-7qpm-85p7), CVSS 7.7) war ein zeitnahes Update der gesamten Server-Komponenten dringend erforderlich.
{{< /admonition >}}

Folgende Updates und Fixes sind in diesem Zuge in den Ports-Tree gewandert:

* **openvoxdb**: `8.12.1` -> `8.18.0` -> `8.18.1`
* **openvox-server**: `8.15.2` -> `8.18.0`
* **openvox**: Der Maintainer `kn@` hatte das Paket bereits auf `8.32.0` aktualisiert. Als ich danach auf die aktuelle Version `8.34.0` updaten wollte, gab es allerdings eine kleine Überraschung beim Abhängigkeits-Check: In `8.32.0` erlaubte die `.metadata` des Ruby-Gems noch problemlos `puppet-resource_api` in Versionen größer als 2. Ab Version `8.33.0` wurde diese Anforderung Upstream jedoch wieder auf `>= 1.9.2, < 2` eingeschränkt (siehe [https://github.com/OpenVoxProject/openvox/pull/733](https://github.com/OpenVoxProject/openvox/pull/733)). Da ich `devel/ruby-puppet-resource_api` in der Zwischenzeit bereits auf `2.1.0` aktualisiert hatte, habe ich die `.metadata` beim Update auf `8.34.0` entsprechend gepatcht. Der Hintergrund für diese Upstream-Einschränkung dürfte vor allem in den All-in-One-Bundles für andere Betriebssysteme liegen, bei denen alle Komponenten gemeinsam paketiert ausgeliefert werden. Ich warte da noch auf Feedback vom Maintainer, dann geht die Version `8.34.0` dann auch in den Ports Tree.
* **openfact & openvox Fixes**: In `openfact` ist ein Fix des Network-Resolver für `gif`- und `gre`-Tunnel in der Queue im Upstream PR [hier](https://github.com/OpenVoxProject/openfact/pull/256), während in `openvox` das Handling von Package-Branches mit Punkten (wie z. B. `8.4`) gefixt wurde.

### Ausflug auf die Alpha-Architektur

Aus reiner Neugier habe ich zwischendurch auch meine alte **Alpha Workstation** vom Staub befreit, um zu schauen, wie es auf dieser Architektur eigentlich um den OpenVox-Client bestellt ist. Damit sich `openvox` dort sauber bauen, installieren und ausführen lässt, brauchten `sysutils/ruby-augeas` und `devel/ruby-ffi` jeweils einen kleinen Patch im `Makefile`: Dort wurde `c++abi` bisher pauschal im `WANTLIB` geführt, darf aber natürlich nur auf Architekturen mit **Clang** als Compiler eingebunden werden – und eben nicht auf GCC-Plattformen wie `alpha`. Das bauen aller Abhängigkeiten hat nur ein paar Tage gedauert, aber danach hatte ich dann einen Alpha Client in meinen OpenVox Server. Die Sinnhaftigkeit der Aktion steht hier übrigens nicht zur Debatte ;)

### Ruby-Gems, PDK, r10k und Puppetboard

Damit OpenVox, das Puppet Development Kit (PDK), `r10k` (das Tool für Git-basiertes Environment- und Modul-Management) sowie `hiera-eyaml-gpg` für verschlüsselte Hiera-Daten reibungslos zusammenspielen, gab es eine ganze Reihe von Gem-Updates – inklusive des Web-Dashboards **Puppetboard**:

* **Puppet- & PDK-Abhängigkeiten**:
  * `archivers/ruby-minitar`: `1.1.0` -> `1.2.0`
  * `net/ruby-msgpack`: `1.8.5` -> `1.8.7`
  * `devel/ruby-puppet-resource_api`: `2.0.1` -> `2.1.0`
  * `devel/ruby-fast_gettext`: `4.1.1` -> `4.2.0`
  * `sysutils/ruby-puppet-syntax`: `7.2.0` -> `7.2.3`
  * `devel/ruby-json-schema` (PDK-Dependency): `6.2.0` -> `6.2.1`
* **Hiera-eyaml-gpg-Abhängigkeit**:
  * `security/ruby-gpgme`: `2.0.26` -> `2.0.27`
* **[r10k](https://github.com/puppetlabs/r10k)-Abhängigkeiten**:
  * `devel/ruby-colored2`: `4.0.3` -> `4.1.0`
  * `www/ruby-faraday`: `2.14.4` -> `2.15.1`
  * `www/ruby-faraday-net_http`: `3.4.4` -> `3.5.0`
* **[Puppetboard](https://github.com/voxpupuli/puppetboard) & Python-Basis**:
  * `www/puppetboard`: `7.0.2` -> `8.1.0`
  * `devel/py-cachelib`: `0.17.0` -> `0.17.2`

{{< figure
    src="/images/puppetboard-in-action.png"
    alt="Puppetboard in Aktion unter OpenBSD"
    caption="Puppetboard in Aktion unter OpenBSD"
    class="ma0 w-75"
>}}

---

## Security-Auditing & Digitale Forensik

Ein weiterer großer Block betraf Werkzeuge aus den Bereichen Web-Security, digitale Forensik, WiFi-Analyse und Software Defined Radio (SDR).

### WPScan und seine Ruby-Abhängigkeiten

**[WPScan](https://wpscan.com/)** ist ein spezialisierter Black-Box-Sicherheitsscanner für WordPress-Installationen, der bekannte Schwachstellen in Core-Versionen, Plugins und Themes aufspürt. Für das Update auf Version `4.2.1` mussten zahlreiche zugrundeliegende Ruby-Gems auf den neuesten Stand gebracht werden:

* `security/wpscan`: `4.1.0` -> `4.2.1`
* `devel/ruby-activesupport`: `8.1.3.1` -> `8.1.4.1`
* `net/ruby-connection_pool`: `3.0.2` -> `3.0.3`
* `www/ruby-ethon`: `0.18.0` -> `0.19.2`
* `www/ruby-xmlrpc`: `0.3.4` -> `0.3.5`
* `www/ruby-ferrum`: `0.18.0` -> `0.21.0`
* `www/ruby-addressable`: `2.9.0` -> `2.10.0`

### Plaso (log2timeline) & Forensik-Bibliotheken

**[Plaso](https://github.com/log2timeline/plaso)** (auch bekannt als das Backend hinter *log2timeline*) ist ein mächtiges Framework für digitale Forensik, das Zeitstempel aus verschiedensten Dateisystemen, Logs und Artefakten extrahiert und zu einer einheitlichen Super-Timeline zusammenführt. Solche komplexen Parser-Suites bringen naturgemäß eigene Bibliotheken für Dateisystem-Abstraktion und Verschlüsselung mit:

* `security/plaso`: `20260720` -> `20261027`
* `security/py-dfdatetime`: `20260730` -> `20261027`
* `security/py-dfvfs`: `20260730` -> `20261027`
* `sysutils/libyal/libcaes`: `20251121` -> `20261028`
* `sysutils/libyal/libfsntfs`: `20260727` -> `20260827`
* `sysutils/afflib3`: `3.7.22` -> `3.7.23`
* `devel/py-parsing`: `3.2.3` -> `3.3.3`

### Kismet, hcxtools und weitere Security-Tools

* **[Kismet](https://kismetwireless.net/)** (`net/kismet` `2025-09-R1` -> `2026-10-R1`): Ein weit verbreiteter Wireless-Network-Detektor, Sniffer und Intrusion-Detection-System für WLAN, Bluetooth, SDR und vieles mehr. Mehr zu Kismet – unter anderem auch zu meinem ursprünglichen Porting-Effort von der steinalten Kismet-Version auf die moderne Codebasis – findet sich in meinen anderen Blogposts [hier](/tags/kismet/). Neben den regulären Änderungen aus den [Release Notes](https://kismetwireless.net/posts/kismet-2026-10-r1/) habe ich Upstream einen neuen Pull Request mit Build-Fixes sowie Anpassungen rund um SUID-Handling beigesteuert ([https://github.com/kismetwireless/kismet/pull/767](https://github.com/kismetwireless/kismet/pull/767)) – und es ist natürlich eine besondere Ehre, in den Release Notes gleich mehrfach namentlich erwähnt zu werden!
  Außerdem wurde ich heute noch von Stuart `sthen@` darauf hingewiesen, dass Kismet beim Bauen unerwünscht `-latomic` aufpickt, sofern `gcc-libs` auf dem System installiert sind. Ein Blick in `configure` bzw. `configure.ac` brachte die Ursache schnell ans Licht: Das Skript prüfte bislang lediglich, *ob* das Linken gegen `-latomic` grundsätzlich funktioniert, aber nicht, ob es überhaupt *notwendig* ist. Entsprechend habe ich `configure.ac` so angepasst, dass zunächst getestet wird, ob ein Programm mit Atomic-Operationen bereits ohne `-latomic` kompiliert und linkt – und `-latomic` nur dann hinzugefügt wird, wenn es tatsächlich gebraucht wird. Upstream PR [hier](https://github.com/kismetwireless/kismet/pull/775).

{{< figure
    src="/images/kismet_in_action.png"
    alt="Kismet in Aktion unter OpenBSD"
    caption="Kismet in Aktion unter OpenBSD"
    class="ma0 w-75"
>}}

Mehrere Data Sources wie Wi-Fi, Zigbee und ADS-B (siehe Screenshot) laufen nun spürbar runder im Parallelbetrieb. Zudem hat Kismet Unterstützung für etliche neue Datenquellen und Hardware-Module erhalten – unter anderem sogar für Gammastrahlungs-Detektoren (eine Liste gibt es [hier](https://www.kismetwireless.net/docs/readme/datasources/datasources/)). Diese neuen Quellen warten bei mir allerdings noch auf einen Praxistest. (**HINT HINT** wer mir Hardware Spenden zukommen lassen möchte, kann sich über Kommentare oder Direktspenden, siehe unten, gern melden ;)

* **[hcxtools](https://github.com/ZerBea/hcxtools)** (`security/hcxtools` `7.1.2` -> `8.0.0`): Eine Sammlung von Werkzeugen zur Konvertierung und Aufbereitung von WLAN-Capture-Dateien für Penetration-Tests.

{{< admonition type=warning title="Breaking Change bei hcxtools 8.0.0" >}}
Mit dem Major-Release `8.0.0` führt `hcxtools` ein geändertes Hash-Dateiformat ein. Die erzeugten Hash-Dateien sind **nicht mehr kompatibel mit älteren Hashcat-Versionen** – hier sollte also gegebenenfalls die gesamte Toolchain aktualisiert werden.
{{< /admonition >}}

* **Weitere Updates aus der Security- & Netzwerk-Ecke**:
  * `security/exploitdb`: `2026-09-12` -> `2026-11-23` (Das lokale Archiv bekannter Exploits und Shellcodes)
  * `security/py-fickling`: `0.1.12` -> `0.1.13` (Statischer Analysator und Decompiler für Python-Pickle-Dateien)
  * `net/proxychains-ng`: `4.17` -> `4.18` (Minor-Bugfix-Release für das Umleiten von TCP-Verbindungen über SOCKS/HTTP-Proxys)
  * `net/udns`: `0.6` -> `0.7` (Asynchrone DNS-Resolver-Bibliothek)

### SDR-Tools (Software Defined Radio)

Passend zum Thema Funkanalyse gab es auch ein kleines Update-Paket rund um **Software Defined Radio (SDR)** – also das Empfangen und Dekodieren verschiedenster Funksignale mithilfe günstiger USB-DVB-T-Sticks auf Basis des RTL2832U-Chipsatzes:

* `comms/rtl-sdr`: `2.0.2` -> `2.0.3` (Die grundlegende Treiber- und Werkzeugsammlung, um RTL2832U-Sticks als universelle SDR-Empfänger unter OpenBSD zu betreiben)
* `comms/readsb`: `3.16.16` -> `3.16.21` (Ein leistungsfähiger ADS-B-Decoder zum Empfangen und Verfolgen von Flugzeug-Transpondersignalen in Echtzeit)

---

## GNUstep, Smart Home & Sonstiges

Abseits von Server-Automatisierung und Security gab es noch ein paar schöne Updates für den Desktop- und Heimgebrauch:

* **SimpleAgenda** (`x11/gnustep/simpleagenda` `0.48` -> `0.50`): Eine schlanke Kalender- und Aufgabenverwaltung für die GNUstep-Desktop-Umgebung. Hier waren einige Anpassungen für die Kompatibilität zwischen `libical` 3 und 4 nötig – die entsprechenden Patches habe ich direkt wieder Upstream geteilt.
* **openHAB** (`misc/openhab` und `misc/openhab-addons` `5.2.1` -> `5.2.4`): Die bekannte herstellerunabhängige Smart-Home-Plattform. Der Patch für dieses Update kam direkt vom Maintainer über die `ports@`-Mailingliste.
* **Ruby HTTPClient** (`www/ruby-httpclient` `2.9.0` -> `2.11.0`): Eine weitere aktualisierte HTTP-Client-Bibliothek für Ruby.

---

## Das war's mal wieder

Dieser Update-Sprint hat mal wieder gezeigt, wie eng verzahnt moderne Software-Ökosysteme sind: Ein einzelner Security-Fix oder das Update eines zentralen Ruby-Gems zieht schnell eine ganze Welle an Folge-Updates nach sich. Besonders erfreulich ist es, wenn dabei nicht nur Pakete unter OpenBSD aktualisiert werden, sondern durch Upstream-Pull-Requests – wie bei Kismet oder SimpleAgenda – auch die Projekte selbst direkt profitieren.

Der Ports-Tree steht bekanntlich niemals still: Die nächsten Ruby-Gems, Python-Pakete und Security-Advisories warten sicher schon um die Ecke. Bis zum nächsten Sprint!

