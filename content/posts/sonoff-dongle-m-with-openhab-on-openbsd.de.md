---
title: "Zigbee Home Automation auf OpenBSD: Sonoff Dongle-M und openHAB"
subtitle: "Zigbee über Ethernet, PoE-Setup und der erfolgreiche Z2M + openHAB Stack auf OpenBSD"
date: 2026-05-14T19:49:28+02:00
lastmod: 2026-05-14T19:49:28+02:00
draft: false
description: "Einrichtung des Sonoff Dongle Max unter OpenBSD mit openHAB und Zigbee2MQTT, inklusive PoE-Tipps und Fehlerbehebung bei Verbindungsproblemen."
license: "MIT"
images: [ "/images/sonoff_openbsd.png", "/images/sonoff-dongle-max.jpeg", "/images/sonoff_dongle_zigbee2mqtt_connection_information.png", "/images/Zigbee2mqtt_frontend.png", "/images/wireshark_zigbee_decrypted.png" ]

tags: ["OpenBSD", "Zigbee", "openHAB", "Zigbee2MQTT", "Sonoff"]
categories: ["Smart Home"]

featuredImage: "/images/sonoff_openbsd.png"
featuredImagePreview: "/images/sonoff_openbsd.png"

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
  images: [ "/images/sonoff_openbsd.png", "/images/sonoff-dongle-max.jpeg", "/images/sonoff_dongle_zigbee2mqtt_connection_information.png", "/images/Zigbee2mqtt_frontend.png", "/i
mages/wireshark_zigbee_decrypted.png" ]

---

In meinem [letzten Post](/zigbee-sniffing-on-openbsd/) haben wir uns das Sniffing von Zigbee-Traffic mit dem älteren [TI CC2531](https://www.ebay.de/itm/404481824304?var=674447321503&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=TICC2531&toolid=10001&mkevt=1) angesehen. Während dieser Stick zum Lernen super ist, ist es jedoch nicht die Hardware, die man für die eigentliche Haussteuerung nutzen möchte. Mein CC2531 eignet sich hervorragend zum Sniffen, da er schon mit der Sniffer-Firmware daherkam – dafür habe ich ihn benutzt und werde ihn auch weiterhin so verwenden.

Meiner Neugier folgend, habe ich mir einen **[Sonoff Dongle Max](https://sonoff.tech/en-de/products/sonoff-dongle-max-zigbee-thread-poe-dongle-dongle-m)** auf [eBay](https://www.ebay.de/itm/388971238285?amdata=enc%3AAQALAAAAoGfYFPkwiKCW4ZNSs2u11xBWFdbfFYp4er2kjyno9qmO593oK669JSSjjTgWvi6tTRNzvYtGUbb3EfOMwwSyITjSDhxkDoFtcCFaXt%2BLbbqQavhhYkKezX99g2LMSYO2N5S0ljwoKg1gnLanibPKbepX68fje0f75JFx9r0FqrNHOksW%2BWM%2BVAIttAcpDD38IEYQS3XIc7r0yK3BY7TAV6w%3D&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Sonoff&toolid=10001&mkevt=1) gekauft, um ihn als vollwertigen **Zigbee-Coordinator** einzusetzen. Im letzten Post konnte ich nur ein paar Zigbee- und andere 802.15.4-Geräte von Nachbarn sehen, aber wo bleibt da der Spaß? Das hat mich neugierig gemacht und mich dazu gebracht, meine eigene Hausautomatisierung zu starten!

{{< admonition type="tip" title="eBay Tipp" >}}
Auf [eBay](https://www.ebay.de/itm/388971238285?amdata=enc%3AAQALAAAAoGfYFPkwiKCW4ZNSs2u11xBWFdbfFYp4er2kjyno9qmO593oK669JSSjjTgWvi6tTRNzvYtGUbb3EfOMwwSyITjSDhxkDoFtcCFaXt%2BLbbqQavhhYkKezX99g2LMSYO2N5S0ljwoKg1gnLanibPKbepX68fje0f75JFx9r0FqrNHOksW%2BWM%2BVAIttAcpDD38IEYQS3XIc7r0yK3BY7TAV6w%3D&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Sonoff&toolid=10001&mkevt=1) findet man den Sonoff Dongle Max zum gleichen Preis wie im offiziellen Shop, kann aber die Funktion "Preisvorschlag senden" nutzen, um das Gerät ein paar Euro günstiger zu bekommen. Bei mir hat es geklappt ;)
{{< /admonition >}}

## Hardware: Der Sonoff Dongle Max

{{< figure
    src="/images/sonoff-dongle-max.jpeg"
    alt="Sonoff Dongle Max"
    caption="Sonoff Dongle Max"
    class="ma0 w-75"
>}}

Der Sonoff Dongle Max ist ein vielseitiges Biest. Im Gegensatz zu den billigen USB-Sticks unterstützt diese Version:
* **USB-Konnektivität**
* **Ethernet mit PoE (Power over Ethernet)**
* **WiFi**

Die "Getting Started"-Dokumentation findet ihr [hier](https://dongle.sonoff.tech/guide/dongle-m/donglem-getting-started/).

Ist der Sonoff Dongle Max via USB verbunden, meldet er sich als serieller Port.

{{< highlight bash >}}
uslcom0 at uhub5 port 2 configuration 1 interface 0 "SONOFF SONOFF Dongle Max MG24" rev 2.00/1.00 addr 5
ucom0 at uslcom0 portno 0: usb1.1.00002.0
{{< /highlight >}}

{{< admonition type="tip" title="Warum PoE?" >}}
Während USB zum Testen ausreicht, zwingt es den Coordinator dazu, direkt neben dem Server zu leben (wahrscheinlich im Keller oder im Rack). Mit **Zigbee over Ethernet** könnt ihr den Dongle zentral im Haus platzieren, indem ihr ein einziges PoE-Kabel für Strom und Daten nutzt. Falls ihr, anders als ich, keinen Switch mit nativem PoE habt, bekommt ihr auch günstige [PoE-Injektoren](https://www.ebay.de/sch/i.html?_nkw=PoE+Injector&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=PoeInjector&toolid=10001&mkevt=1) bei eBay.
{{< /admonition >}}

### Die "Smart IP" Kopfschmerzen
Eine kurze Warnung zu den Werkseinstellungen: Der Dongle nutzt "Smart IP". Er merkt sich die allererste IP, die er per DHCP erhält, und klebt förmlich an ihr, selbst wenn der DHCP-Server später versucht, ein anderes statisches Lease zuzuweisen.

Falls ihr Probleme habt, das Gerät nach der Einrichtung eines statischen Leases zu erreichen, erinnert ihr euch hoffentlich an die ursprüngliche IP oder müsst euch mit dem internen WiFi-AP verbinden, um die Einstellungen manuell von "Smart IP" auf Standard-DHCP umzustellen.

Das hat mir einiges an Kopfzerbrechen bereitet ;)

Sobald das geklärt ist, könnt ihr euren Browser auf `http://<Hostname oder IP-Adresse>` richten, um zum Web-Frontend des Sonoff Dongle Max zu gelangen.

### Verbindungsinformationen finden
Um openHAB oder Zigbee2MQTT zu konfigurieren, benötigt ihr die spezifischen Port-Strings aus dem Web-Interface des Dongles. Navigiert im Menü links zu **Z2M&ZHA -> Zigbee2MQTT**. Dort könnt ihr die Sektionen **USB Connection** und **TCP Connection** ausklappen, um die exakten Pfade und Portnummern für eure Konfigurationsdateien zu erhalten.

{{< figure
    src="/images/sonoff_dongle_zigbee2mqtt_connection_information.png"
    alt="Zigbee2MQTT Verbindungsinfos im Sonoff Web UI"
    caption="Zigbee2MQTT Verbindungsinformationen im Sonoff Web UI"
    class="ma0 w-75"
>}}

{{< admonition type="tip" title="Verbindungs-Info" >}}
Für eine serielle Verbindung über USB ist unter OpenBSD das zu verwendende Gerät typischerweise `/dev/cuaU0` oder ähnlich. Bei der TCP-Verbindung stellt sicher, dass ihr den richtigen Hostnamen verwendet oder bleibt bei der IP-Adresse.
{{< /admonition >}}

## Einrichtung von openHAB auf OpenBSD

Die Installation von openHAB auf OpenBSD ist dank der verfügbaren Pakete unkompliziert. Sobald die Pakete installiert sind, muss nur noch der Dienst gestartet werden.

{{< highlight bash >}}
pkg_add -i openhab%5 openhab-addons%5
rcctl start openhab
{{< /highlight >}}

Nachdem der Dienst gestartet ist, öffnet `http://localhost:8080` (oder die IP eures Servers) im Browser. Ihr werdet vom Setup-Assistenten begrüßt, wo ihr euren ersten **Administrator**-Account erstellt. Sobald ihr eingeloggt seid, ist der erste Schritt der Weg zu **Settings > Add-ons**, um das **Zigbee Binding** zu installieren.

{{< admonition type="info" title="Neu bei openHAB?" >}}
Wenn ihr bei der Hausautomatisierung und openHAB genauso neu seid wie ich, empfehle ich dringend, die offizielle [openHAB Dokumentation](https://www.openhab.org/docs/) zu lesen. Sie erklärt die Kernkonzepte wie Things, Items und Channels hervorragend, was am Anfang etwas überwältigend sein kann! Schaut euch daher zuerst die [Concepts](https://www.openhab.org/docs/concepts/) sowie das [Getting Started Tutorial](https://www.openhab.org/docs/tutorial/) an.
{{< /admonition >}}

## Erster Versuch: Der direkte Weg über das Zigbee Binding

Nach dem Login war mein erster Instinkt, es einfach zu halten und das native Binding zu nutzen:

1. Gehe zu **Settings > Add-ons** und installiere das **Zigbee Binding**.
2. Navigiere zu **Settings > Things**, klicke auf das blaue **+** und wähle das **Zigbee Binding**.
3. Wähle **Ember Coordinator** aus der Liste (der richtige Treiber für den Sonoff Dongle-M).
4. In der Konfiguration setzte ich den Port auf: `tcp://10.0.0.210:6638`.

Zusätzlich konfigurierte ich die Baudrate auf `115200` und Flow Control auf `None`. Doch selbst nach dem Speichern blieb das Thing hartnäckig offline.

Um zu sehen, was unter der Haube passierte, erhöhte ich das Log-Level in der Karaf-Konsole:

{{< highlight bash >}}
# Verbindung zu Karaf (Standard-Passwort: habopen)
ssh -p 8101 openhab@localhost
log:set DEBUG org.openhab.binding.zigbee
log:set DEBUG com.zsmartsystems.zigbee
{{< /highlight >}}

Das Log offenbarte das Problem:

{{< highlight text >}}
[DEBUG] [nding.zigbee.serial.ZigBeeSerialPort] - Connecting to serial port [tcp://10.0.0.210:6638] at 115200 baud...
[DEBUG] [nding.zigbee.serial.ZigBeeSerialPort] - No communication ports found, cannot connect to [tcp://10.0.0.210:6638]
[ERROR] [zigbee.dongle.ember.ZigBeeDongleEzsp] - EZSP Dongle: Unable to open serial port
{{< /highlight >}}

Ein Blick in das `README` des OpenBSD-Ports gab einen entscheidenden Hinweis zum Java-Seriell-Zugriff:
> "The example of passing `-Dgnu.io.rxtx.SerialPorts` uses the rxtx library, which is currently unsupported on OpenBSD. However, 100% Java implementations such as PureJavaComm and jSerialComm are known to work."

Das `README` bezieht sich auf physische serielle Ports, nicht auf TCP. Da die TCP-Methode jedoch sofort fehlschlug und ich den Hinweis auf die fehlende `rxtx`-Unterstützung sah, habe ich einen lokalen seriellen Anschluss gar nicht erst versucht. Zeit für einen robusteren architektonischen Ansatz.

## Der Gewinner-Stack: Zigbee2MQTT + Mosquitto

Anstelle einer direkten Verbindung wechselte ich zu einem entkoppelten Stack:
**Sonoff (TCP) <-> Zigbee2MQTT <-> Mosquitto (MQTT) <-> openHAB.**

### 1. Tools installieren
{{< highlight bash >}}
pkg_add -i mosquitto zigbee2mqtt
{{< /highlight >}}

### 2. Zigbee2MQTT konfigurieren
Editiert eure `/etc/zigbee2mqtt/configuration.yaml`. Beachtet die Einstellung `adapter: ember`, die für den MG24-Chip im Dongle-M erforderlich ist.

{{< highlight yaml >}}
version: 5
homeassistant:
  enabled: true
frontend:
  enabled: true
  host: 127.0.0.1
  port: 8081
mqtt:
  base_topic: zigbee2mqtt
  server: mqtt://localhost
serial:
  port: tcp://10.0.0.210:6638
  baudrate: 115200
  adapter: ember
  disable_led: false
  rtscts: false
advanced:
  channel: 25
  log_level: debug
  log_directory: /var/log/zigbee2mqtt
  log_file: zigbee2mqtt_%TIMESTAMP%.log
  log_rotation: true
  log_output:
    - file
  network_key: GENERATE
  pan_id: GENERATE
  ext_pan_id: GENERATE
availability:
  enabled: true
{{< /highlight >}}

Wenn ihr kein PoE zur Verfügung habt oder den Dongle lieber direkt über USB anschließen wollt, ändert eure `serial:`-Konfiguration wie folgt:

{{< highlight yaml >}}
serial:
  port: /dev/cuaU0
  adapter: ember
  baudrate: 115200
  disable_led: false
  rtscts: false
{{< /highlight >}}

{{< admonition type="info" title="USB Berechtigungen auf OpenBSD" >}}
Wenn ihr die USB-Verbindung nutzt, muss der Benutzer `_z2m` die Berechtigung haben, auf das serielle Funkgerät (typischerweise `/dev/cuaU0`) zuzugreifen. Unter OpenBSD macht ihr das, indem ihr den Benutzer zur Gruppe **dialer** hinzufügt:

{{< highlight bash >}}
usermod -G dialer _z2m
{{< /highlight >}}
{{< /admonition >}}

{{< admonition type="info" title="Zigbee2MQTT Dokumentation" >}}
Ähnlich wie die openHAB-Dokumentation ist auch die [Zigbee2MQTT-Dokumentation](https://www.zigbee2mqtt.io/guide/getting-started/) definitiv einen Blick wert.
{{< /admonition >}}

### Mosquitto konfigurieren
Für eine einfache lokale Installation ist die Standardkonfiguration ausreichend; es sind keine unmittelbaren Änderungen erforderlich, um loszulegen.

{{< admonition type="info" title="Mosquitto erkunden" >}}
Während die Standardeinstellungen out-of-the-box funktionieren, ist die offizielle [Mosquitto-Dokumentation](https://mosquitto.org/documentation/) eine exzellente Ressource, falls ihr später tiefer in Sicherheitsaspekte, Bridging oder komplexeres Message-Routing eintauchen wollt.
{{< /admonition >}}

### 3. Dienste starten
{{< highlight bash >}}
rcctl start mosquitto
rcctl start zigbee2mqtt
{{< /highlight >}}

### Das Zigbee2MQTT Frontend
Da wir das `frontend` in der Konfigurationsdatei aktiviert haben, könnt ihr euer Netzwerk direkt über den Browser verwalten. Standardmäßig ist es unter `http://127.0.0.1:8081` erreichbar. Dieses Interface ist unglaublich hilfreich beim Koppeln neuer Geräte, zur Ansicht der Netzwerkkarte und zur Überwachung des Status eurer Sensoren.

{{< figure
    src="/images/Zigbee2mqtt_frontend.png"
    alt="Das Zigbee2MQTT Web Frontend"
    caption="Das Zigbee2MQTT Web Frontend"
    class="ma0 w-75"
>}}

## Integration in openHAB

Sobald Zigbee2MQTT läuft, ist die Integration nahtlos:
1. Installiere das **MQTT Binding** in openHAB.
2. Füge ein neues **MQTT Broker** Thing hinzu, das auf `127.0.0.1` zeigt.
3. Da wir `homeassistant: true` in Z2M aktiviert haben, sollten eure Geräte automatisch in der openHAB Inbox erscheinen, sobald sie gekoppelt werden!

## Bonus: Sniffing des neuen Netzwerks
Auch wenn ich das System jetzt produktiv nutze, will der "Sniffer" in mir immer noch die Pakete sehen. Um den Traffic eures neuen Netzwerks in Wireshark zu entschlüsseln, benötigt ihr den Network Key.

Beim ersten Start aktualisiert Zigbee2MQTT den `GENERATE`-String in eurer `/etc/zigbee2mqtt/configuration.yaml`, was dann etwa so aussieht:

{{< highlight yaml >}}
  network_key:
    - 104
    - 73
    - 101
    - 15
    - 20
    - 56
    - 102
    - 28
    - 132
    - 77
    - 211
    - 116
    - 109
    - 134
    - 9
    - 20
{{< /highlight >}}

{{< admonition type="danger" title="Schützt euren Network Key!" >}}
Behandelt euren **Network Key** wie euer sensibelstes Passwort. Wenn jemand diesen Key erhält und sich in physischer Reichweite eures Hauses befindet, kann er:
* **Den gesamten Traffic entschlüsseln:** Jedes Sensor-Ereignis, jeden Schaltzustand und jedes Datenpaket im Netzwerk mitlesen.
* **Befehle einschleusen:** Eure Lichter, Schlösser oder andere Zigbee-Geräte ohne Erlaubnis fernsteuern.
* **Passiv sniffen:** Da Zigbee ein Mesh-Protokoll ist, muss man nicht einmal mit dem WLAN verbunden sein; ein einfacher USB-Sniffer reicht aus.

Nutzt diesen Key nur für eure eigene private Fehlersuche in Wireshark und postet ihn **niemals** in einem öffentlichen Blog oder Forum!
{{< /admonition >}}

{{< highlight bash >}}
# Euer Key aus der configuration.yaml
# - 104 - 73 - 101 - 15 - 20 ...

printf '%02x' 104 73 101 15 20 56 102 28 132 77 211 116 109 134 9 20 && echo
# Output: 6849650f1438661c844dd3746d860914
{{< /highlight >}}

Mit meinem [TI CC 2531](https://www.ebay.de/itm/404481824304) kann ich nun `whsniff` nutzen (welches es erst heute in den OpenBSD-Ports-Tree geschafft hat), um IEEE802.15.4-Traffic in Wireshark einzuspeisen:

{{< highlight bash >}}
sudo pkg_add -i whsniff wireshark
sudo whsniff -c 25 | wireshark -k -i -
{{< /highlight >}}

Sobald Wireshark läuft, nehmt den Output-String von oben und fügt ihn unter **Wireshark -> Preferences -> Protocols -> Zigbee -> Keys** hinzu. Jetzt könnt ihr eure Home-Automation-Befehle im Klartext durch die Luft fliegen sehen!

{{< figure
    src="/images/wireshark_zigbee_decrypted.png"
    alt="Entschlüsselten Zigbee-Traffic in Wireshark sniffen"
    caption="Entschlüsselten Zigbee-Traffic in Wireshark sniffen"
    class="ma0 w-75"
>}}

{{< admonition type="info" title="Adressen dekodieren" >}}
Beim Blick in die Wireshark-Logs werdet ihr viele Pakete sehen, die von oder zu der Adresse **`0x0000`** gesendet werden. In der Zigbee-Welt ist dies die standardisierte Kurzadresse für den **Network Coordinator** (in meinem Fall der Sonoff Dongle Max).
{{< /admonition >}}

Damit kann ich meine Befehle beobachten – diesmal zu meinen eigenen Bedingungen!

## Fazit

Diesen Stack auf OpenBSD zum Laufen zu bringen, war eine kleine Herausforderung, aber eine lohnende. Zum Abschluss dieses initialen Setups erinnert mich ein Abschnitt in der [openHAB Einleitung](https://www.openhab.org/docs/#what-you-need-to-know-before-you-start) daran, was wirklich zählt:

> **What You Need to Know Before You Start**
>
> When home automation just seems to work, it is always the result of hard work. Home automation is fascinating and requires a considerable investment of your time. Here are some key considerations especially for new users. To be successful, you will need to:
>
> * Start slowly and proceed one step at a time
> * Be prepared to learn
> * Remain flexible in how you want to achieve your goal
> * Celebrate all the small successes

Genau das tue ich hier, und wahrscheinlich auch in einer Reihe von folgenden Beiträgen. Dass der Coordinator mit dem Stack auf OpenBSD spricht, ist ein bedeutender erster Schritt und ein Erfolg für mich – einer, den ich definitiv feiern werde.

Die Dokumentation lässt uns zudem mit einer letzten Warnung zurück:

> "Lastly, be prepared to start a new hobby: home automation."

Das ist zweifellos wahr, aber zum Glück überschneidet es sich bei mir stark mit meinen anderen Hobbys, was es schließlich zu einer sehr natürlichen Erweiterung macht!

Bleibt dran – die Sensoren sind bereits unterwegs.

