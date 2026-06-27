---
title: "NCC-1701 im Smart Home: Präsenzerkennung mit openHAB und UniFi"
subtitle: "Wie eine sensorbasierte Brücken-Phalanx den openHAB-Maschinenraum zähmt."
date: 2026-06-27T10:49:41+02:00
lastmod: 2026-06-27T10:49:41+02:00
draft: false
description: "Eine Anleitung zur stabilen Präsenzerkennung mit dem openHAB Network- und UniFi-Binding. Umgehe den UI-Equipment-Stolperstein und bringe deine Crew via At Home Card fehlerfrei auf das Dashboard."
license: "MIT"
images: [ "/images/Crew_On_Deck.png", "/images/pingable_device.png", "/images/unifi_client.png", "/images/Group_kirk.png" ]

tags: ["StarTrek", "openHAB", "SmartHome", "UniFi", "Präsenzerkennung"]
categories: [ "Smart Home" ]

featuredImage: "/images/Crew_On_Deck.png"
featuredImagePreview: "/images/Crew_On_Deck.png"

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
  images: [ "/images/Crew_On_Deck.png", "/images/pingable_device.png", "/images/unifi_client.png", "/images/Group_kirk.png" ]
  # ...
---


# "Alle an Bord, Scotty?“ 

Wer [openHAB](https://www.openhab.org/) kennt, weiß: Die Lernkurve ist steil, aber die Möglichkeiten sind extrem vielseitig. Schnell wird einem klar, dass starre Zeitpläne ein Haus noch nicht *smart* machen. Ein echtes Smart Home muss wissen, wer an Bord ist, um die Lebenserhaltungssysteme (Heizung, Licht, Alarmanlage) sinnvoll zu steuern.

Als alter Trekkie stand das Design für mein Dashboard schnell fest: Die klassische **TOS-Crew (The Original Series)** sollte unsere fünfköpfige Familie abbilden. Wenn ein Crewmitglied an Bord ist, leuchtet das Avatar-Bild im UI-Widget farbig. Verlässt jemand das Schiff, schaltet die Anzeige auf *Grayscale* (Graustufen) und signalisiert den "Away Mission"-Status. 

Was einfach klingt, fühlte sich im openHAB-Maschinenraum schnell so an, als hätte jemand eine Kiste Tribbles an Bord gebracht: Nicht alles funktionierte gleich "out of the box". Mit etwas Geduld, Trial and Error auf der Suche nach den korrekten Optionen und deren Einstellungen, ließ sich die Kommunikator-Phalanx am Ende ganz passabel kalibrieren. Ich stehe noch relativ am Anfang meiner Smart-Home-Odyssee und teile hier meine Erfahrungen mit der IP- und WLAN-Erkennung sowie der korrekten Gruppenkonfiguration.

Werfe gerne auch einen Blick auf [/categories/smart-home/](/categories/smart-home/) f\xc3\xbcr meine anderen Artikel rund um [openHAB](https://www.openhab.org/) auf [OpenBSD](https://www.openbsd.org/).

---

## 1. Vorbereitungen am Handy (Android & iOS)

Bevor openHAB nach Geräten suchen kann, müssen die Smartphones netzwerkfreundlich konfiguriert werden. Heutige Betriebssysteme sind auf maximalen Datenschutz getrimmt – zwei Standard-Features stehen einer dauerhaften Erkennung im WLAN jedoch im Weg.

### iOS (iPhone)
* **Private WLAN-Adresse deaktivieren:** Apple würfelt standardmäßig bei jeder Verbindung eine neue, zufällige MAC-Adresse aus. Für ein stabiles Tracking im Heimnetzwerk ist das unbrauchbar.
  * *Weg:* `Einstellungen` -> `WLAN` -> Auf das blaue **(i)** neben dem Heim-WLAN tippen -> **Private WLAN-Adresse ausschalten**.
* **Statische IP-Adresse:** Sorge im Router (z. B. OpenBSD, UniFi oder FritzBox) dafür, dass dem iPhone über den DHCP-Server immer die gleiche IP-Adresse zugewiesen wird.

{{< admonition type="note" title="Hintergrund: Apples MAC-Adressen-Modi im Vergleich" open=true >}}
Apple unterscheidet in den WLAN-Einstellungen seit neueren iOS-Versionen zwischen drei Stufen für die MAC-Adresse:

1. **Rotierend (Rotating / Ein):** Die MAC-Adresse ändert sich regelmäßig, selbst im selben Netzwerk. Macht lokales Tracking unmöglich.
2. **Fest (Fixed / Privat):** Das iPhone nutzt eine zufällige MAC-Adresse, diese bleibt aber *speziell für dieses eine WLAN* dauerhaft gleich.
3. **Aus (Off):** Das iPhone sendet seine echte, im Chip eingebrannte Hardware-MAC-Adresse an den Router.

#### Vor- und Nachteile im Heimnetzwerk: "Aus" (Off) vs. "Fest" (Fixed)

| Modus | Vorteile | Nachteile |
| :--- | :--- | :--- |
| **Aus (Off)** | • Absolute Stabilität bei Bindings.<br>• Perfekt für strikte MAC-Filter im Router.<br>• Keine Überraschungen nach iOS-Updates. | • Schaltet den globalen Tracking-Schutz für dieses Netz komplett ab.<br>• Ein Klick weniger Datenschutz im eigenen LAN. |
| **Fest (Fixed)** | • Guter Kompromiss aus Datenschutz (Router sieht echte MAC nicht) und statischem Tracking. | • Nach einem "Netzwerk zurücksetzen" am iPhone wird eine neue feste MAC generiert; openHAB verliert das Gerät. |

**Empfehlung für das Smart Home:** Da das eigene Heimnetzwerk vertrauenswürdig ist, ist das komplette Abschalten (**Aus / Off**) die stressfreiste Variante für den Captain. Wer maximale Datensparsamkeit will, nutzt **Fest (Fixed)**, muss den Client (MAC Adresse) bei einem Geräte-Reset neu konfigurieren.
{{< /admonition >}}

### Android
* **Zufällige MAC-Adresse deaktivieren:** Auch Google nutzt standardmäßig randomisierte Adressen.
  * *Weg:* `Einstellungen` -> `Netzwerk & Internet` -> `Internet` -> Zahnrad neben dem Heim-WLAN -> `Datenschutz` -> **Geräte-MAC verwenden** wählen.

---

## 2. Die Brücken-Crew: Wer ist wer?

Um das System im openHAB Semantic Model sauber aufzubauen, habe ich die Familie in fünf bekannte Charaktere aufgeteilt:

* **Dad:** `Captain Kirk` (Behält den Überblick)
* **Mom:** `Uhura` (Hält die Kommunikationskanäle der Familie zusammen)
* **Kind 1:** `Spock` (Der Logiker, der fasziniert vor dem Bildschirm sitzt)
* **Kind 2:** `Scotty` (Unser Energiebündel für den Maschinenraum)
* **Kind 3:** `Pille / McCoy` (Der Jüngste, der gerne mal meutert)

---

## 3. Lokales Tracking: IP-Ping vs. UniFi WiFi Client

Für das Tracking im lokalen Netzwerk habe ich zwei Methoden getestet.

### Methode A: Das „Pingable Network Device“ (Network Binding)
Diese Methode nutzt das "Network Binding" das vorher installiert sein sollte.
openHAB sendet hierbei in festen Abständen einen kurzen Netzwerk-Ping an die feste IP-Adresse des Smartphones. Antwortet das Handy, gilt es als `ON`.

#### Code für's Pingable Device
Hier ein Beispiel für Kapitän Kirk, die anderen Crewmitglieder dann äquivalent:
{{< highlight yaml >}}
Thing network:pingdevice:KirkCommunicator "Kirk Communicator Ping" [hostname="192.168.0.42", macAddress="11:22:33:44:55:66", refreshInterval=60000, retry=1, timeout=
5000, useIOSWakeUp=true, useArpPing=true, useIcmpPing=true] {
        Channels:
                Type online : online "Online"
                Type latency : latency "Latency"
                Type lastseen : lastseen "Last Seen"
}
{{< /highlight >}}

* **Erfahrung:** Schnell eingerichtet und funktioniert mit jedem Router. **Das Problem:** Geht ein Smartphone (besonders iPhones) in den Deep Sleep, schaltet es das WLAN-Modul temporär ab. openHAB denkt dann, man hat das Haus verlassen. Es kommt zu Fehlalarmen.

{{< figure
    src="/images/pingable_device.png"
    alt="24h Erreichbarkeit des Pingable Devices"
    caption="24h Erreichbarkeit des Pingable Devices"
    class="ma0 w-75"
>}}


### Methode B: Das UniFi Wi-Fi Client Device (UniFi Binding)
Diese Methode nutzt das "Unifi Binding" das vorher installiert sein sollte.
Da ich ein Ubiquiti UniFi-Netzwerk nutze, ist das die deutlich solidere Variante. openHAB fragt hierbei direkt den UniFi-Controller ab, ob die spezifische **MAC-Adresse** des Handys im WLAN angemeldet ist.

#### Code für's Pingable Device 
Hier ein Beispiel für Kapitän Kirk, die anderen Crewmitglieder dann äquivalent:
{{< highlight yaml>}}
Thing unifi:wirelessClient:KirkCommunicator "Kirks Communicator" (unifi:controller:e5b4544ee2) [cid="11:22:33:44:55:66", considerHome=180] {
        Channels:
                Type online : online "Online"
                Type name : name "Name"
                Type hostname : hostname "Hostname"
                Type site : site "Site Name"
                Type macAddress : macAddress "MAC Address"
                Type ipAddress : ipAddress "IP Address"
                Type uptime : uptime "Uptime"
                Type lastSeen : lastSeen "Last Seen"
                Type blocked : blocked "Blocked"
                Type experience : experience "Experience"
                Type guest : guest "Guest"
                Type ap : ap "Access Point"
                Type essid : essid "Wireless Network"
                Type rssi : rssi "Received Signal Strength Indicator"
                Type wirelessCmd : cmd "Wireless Command"
                Type reconnect : reconnect "Reconnect"
}
{{< /highlight >}}

* **Erfahrung:** Ein echter Sprung nach vorne. Das iPhone meldet sich beim Annähern an das Haus im WLAN an, noch *bevor* es eine IP-Adresse bezogen hat. Der Controller weiß, dass das Handy im Haus ist, selbst wenn es im Standby schläft. Über das Binding lässt sich zudem auslesen, an welchem Access Point (z. B. im Maschinenraum oder Zehn-Vorne) sich das Crewmitglied gerade aufhält.

{{< figure
    src="/images/unifi_client.png"
    alt="24h Erreichbarkeit des WiFi clients"
    caption="24h Erreichbarkeit des WiFi clients"
    class="ma0 w-75"
>}}

Viel Besser!

---

## 4. Das openHAB-Rätsel: Gruppen- & Item-Konfiguration

Hier lag der größte Stolperstein. Ich hatte für jedes Crewmitglied ein "Equipment" (eine Gruppe) angelegt. Darin befand sich die jeweilige Items für den Online-Status des Kommunikators via Ping und Unifi Client Tracking, sowie der Name des Access Points, an dem der Kommunikator des Crewmitgliedes angemeldet ist.

Obwohl die Items im System korrekt den Status `Online` anzeigten, blieben die übergeordneten Familien-Equipments im Status `NULL` (uninitialisiert) hängen. Das Dashboard-Widget blieb grau. Warum? Eine normale openHAB-Gruppe weiß standardmäßig nicht, wie sie den Status ihrer Unter-Items aggregieren soll. Wir müssen openHAB explizit sagen, dass die Gruppe den `Switch`-Status überwachen und mittels einer logischen Funktion zusammenfassen soll.

Die logische Definition aller Gruppen und Items sieht strukturell wie folgt aus:

{{< highlight text >}}
Group:Switch:OR(ON, OFF) Crew "Crew" <f7:person_3>

Group:Switch:OR(ON, OFF) Kirk "Kirk" <man_3> (Crew) ["Equipment"]
Group:Switch:OR(ON, OFF) Uhura "Uhura" <woman_3> (Crew) ["Equipment"]
Group:Switch:OR(ON, OFF) Spock "Spock" <boy_3> (Crew) ["Equipment"]
Group:Switch:OR(ON, OFF) Scotty "Scotty" <boy_4> (Crew) ["Equipment"]
Group:Switch:OR(ON, OFF) McCoy "McCoy" <boy_5> (Crew) ["Equipment"]
{{< /highlight >}}

{{< admonition type="warning" title="Der UI-Stolperstein: Equipment vs. Switch" open=true >}}
Hier stößt man auf eine Design-Einschränkung der openHAB MainUI: 
* Erstellt man ein Item vom Typ `Group` und taggt es semantisch als **Equipment**, erlaubt das UI standardmäßig *keine* Auswahl einer Aggregationsfunktion.
* Ändert man den Gruppentyp im UI stattdessen hart auf **Switch**, um die Aggregatfunktion freizuschalten, lassen sich die darunterliegenden Crewmitglieder-Items oft nicht mehr sauber zuordnen.

**Der Ausweg:** Man belässt den Gruppentyp im UI auf *Equipment* und fügt den spezifischen `group`-Unterblock mit der gewünschten Logik nachträglich manuell im **Code-Tab (YAML)** ein.

**Hinweis zum Aggregations-Verhalten:** Da die Gruppe im YAML-Block explizit als `type: Switch` definiert ist, berücksichtigt die Aggregations-Engine von openHAB auch nur Unter-Items vom Typ `Switch` (wie den Ping- und UniFi-Online-Status). Alle anderen Typen – wie der `String` des `Kirk_Subspace_Node` (Access Point Name) – werden bei der Statusberechnung automatisch ignoriert. Das verhindert Formatkonflikte und hält die Tracking-Logik sauber.

**Warum OR(ON, OFF) statt AND(ON, OFF)?**
Während eine `AND`-Verknüpfung voraussetzen würde, dass *sowohl* der Ping *als auch* das UniFi-Binding gleichzeitig `ON` melden, reicht uns beim Präsenz-Tracking ein logisches `OR`. Das bedeutet: Sobald mindestens ein Sensor meldet, dass der Kommunikator in Reichweite ist, gilt das Crewmitglied als an Bord. Erst wenn beide Kanäle unabhängig voneinander auf `OFF` gehen, schaltet die gesamte Gruppe auf abwesend. Das fängt temporäre Aussetzer (wie den Deep Sleep beim Ping) perfekt ab.

**Die Crew-Gruppe: Warum auch hier ein OR die beste Wahl ist**
Für die Hauptgruppe `Crew` nutzen wir ebenfalls eine `OR(ON, OFF)`-Aggregation. Das hat einen einfachen, praktischen Grund: Die Gruppe soll den globalen Status „Jemand zu Hause“ widerspiegeln (z. B. um die Alarmanlage zu deaktivieren, sobald die erste Person das Haus betritt). Ein `AND` wäre hier unbrauchbar, da die Gruppe dann nur auf `ON` springen würde, wenn ausnahmslos *alle* Familienmitglieder gleichzeitig an Bord sind – das System würde das Haus also fälschlicherweise als leer betrachten, selbst wenn vier von fünf Personen auf der Brücke sitzen.


{{< /admonition >}}

### YAML-Code für die Equipment Gruppe "Kirk"
{{< highlight yaml >}}
version: 1
items:
  Kirk:
    type: Group
    label: Kirk
    icon: man_3
    groups:
      - Crew
    tags:
      - Equipment
    group:
      type: Switch
      function: OR
      parameters:
        - ON
        - OFF
{{< /highlight >}}

### DSL-Code der Items in der Kirk Gruppe:
{{< highlight text >}}
Switch Kirk_Online "Communicator Online" <f7:antenna_radiowaves_left_right> (Kirk) ["Presence", "Status"] {
  channel="unifi:wirelessClient:KirkCommunicator:online",
  stateDescription=" " [options="ON=Online,OFF=Offline", readOnly=true]
}
Switch Kirk_Communicator_Ping "Communicator Ping" <f7:waveform_path_ecg> (Kirk) ["Presence", "Status"] {
  channel="network:pingdevice:KirkCommunicator:online",
  stateDescription=" " [options="ON=Online,OFF=Offline", readOnly=true]
}
String Kirk_Subspace_Node "Subspace Node" <f7:wifi> (Kirk) ["Point"] {
  channel="unifi:wirelessClient:KirkCommunicator:ap",
  stateDescription=" " [readOnly=true]
}
... (Andere Crewmitglieder äquivalent aufgebaut)
{{< /highlight >}}

Das Ergebnis sieht man dann hier:

{{< figure
    src="/images/Group_kirk.png"
    alt="Kapitän Kirk ist auf der Enterprise"
    caption="Kapitän Kirk ist auf der Enterprise"
    class="ma0 w-75"
>}}

---

## 5. Das visuelle Highlight: Die „At Home Card“ auf der Brücke

Nachdem die Logik im Maschinenraum fehlerfrei läuft, bringen wir die Daten auf den Hauptbildschirm. Als visuelle Schnittstelle nutze ich das beliebte **At Home Card**-Widget aus dem openHAB Community Marketplace. 

Dieses Widget liest die übergeordnete Gruppe (in unserem Fall `Crew`) aus, erkennt automatisch alle darin enthaltenen Equipments (unsere Crewmitglieder) und wechselt je nach deren Status (`ON` oder `OFF`) das Avatar-Bild zwischen Farbe und Graustufen.

### Schritt 1: Die Avatare im Dateisystem hinterlegen
Damit das Widget die Bilder der TOS-Crew laden kann, müssen sie auf dem openHAB-Server im lokalen HTML-Verzeichnis abgelegt werden. 

{{< admonition type="tip" title="Wichtig bei den Dateinamen" open=true >}}
Das Widget sucht im Ordner standardmäßig nach Bildern, die exakt so heißen wie der **Name des jeweiligen Items** (nicht das Label!). Wenn dein Item `Kirk` heißt, muss das Bild folglich `Kirk.png` (bzw. `.jpg`) heißen. Achte penibel auf Groß- und Kleinschreibung.
{{< /admonition >}}

Kopiere die Dateien auf den openHAB-System und erstelle den passenden Unterordner:

Pfad auf dem Server: `/etc/openhab/html/presence/`

Dort lädst du die fünf Bilder der Besatzung hoch:
* `Kirk.png`
* `Uhura.png`
* `Spock.png`
* `Scotty.png`
* `McCoy.png`

Über den lokalen Webserver von openHAB sind diese Bilder nun unter der URL `http://<deine-openhab-ip>:8080/static/presence/Kirk.png` erreichbar.

### Schritt 2: Das Widget installieren und konfigurieren
1. Gehe im MainUI auf **Add-on Store**
2. Suche nach **At Home Card** und klicke auf **Install**.
3. Erstelle eine neue Pages-Ansicht oder bearbeite ein bestehendes Layout.
4. Füge das Widget hinzu und wechsle in dessen **Code-Tab**, um es an unsere Crew anzupassen.

Hier ist die YAML-Konfiguration für das Widget:

{{< highlight yaml >}}
component: widget:at_home_card
config:
  family: Crew
  title: On Deck
  bggradienttopcolor: rgba(253, 242, 201, 1)
  bggradientbottomcolor: rgba(245, 220, 160, 1)
{{< /highlight >}}

Über den Parameter `family: Crew` weiß das Widget sofort, wo es nach den Avataren suchen muss. Sobald du die Seite speicherst, siehst du deine Brücken-Crew übersichtlich aufgereiht. Crewmitglieder an Board sind bunt, Mitglieder auf Mission sind grau.

{{< figure
    src="/images/Crew_On_Deck.png"
    alt="Alle an Board?"
    caption="Alle an Board?"
    class="ma0 w-75"
>}}

## 5. Fazit & Ausblick

Die Präsenzerkennung über das lokale WLAN und das UniFi-Tracking läuft nun stabil. Die TOS-Crew wird auf dem Dashboard fehlerfrei überwacht. 

Man muss sich jedoch bewusst sein: Das WLAN-Tracking ist nur **ein** Baustein im gesamten System. Es deckt primär den Nahbereich ab. Wenn man das Haus verlässt, dauert es systembedingt meist einige Minuten, bis der UniFi-Controller ein Gerät offiziell als abgemeldet markiert.

{{< admonition type="note" title="Zukünftige Ausbaustufen" open=true >}}
Um die Präsenzerkennung absolut redundant zu machen, plane ich folgende Erweiterungen:
1. **[GPS-Tracking via OwnTracks](https://www.openhab.org/addons/bindings/gpstracker/):** Damit erkennt openHAB, wenn ich mich dem Haus auf 500 Meter nähere – perfekt, um die Heizung schon hochzufahren.
2. **Bluetooth Beacons (BLE):** Günstige Bluetooth-Sender am Schlüsselbund, die von ESP32-Empfängern im Haus registriert werden, um eine raumspezifische Erkennung im Innenraum zu ermöglichen.
{{< /admonition >}}

Die Reise hat gerade erst begonnen, aber ein wichtiger Meilenstein ist geschafft. Das System weiß endlich, wer an Bord ist.

*Wie habt ihr eure Präsenzerkennung gelöst? Setzt ihr rein auf WLAN oder kombiniert ihr es mit GPS? Schreibt es mir gerne in die Kommentare!*


