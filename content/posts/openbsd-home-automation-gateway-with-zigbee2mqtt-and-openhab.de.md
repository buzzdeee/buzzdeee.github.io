---
title: "OpenBSD Home-Automation-Gateway mit Zigbee2MQTT und openHAB"
subtitle: "Ein lautloses, isoliertes IoT-Gateway mit Zigbee2MQTT, Mosquitto und openHAB auf lüfterloser x86-Hardware"
date: 2026-06-11T16:52:20+02:00
lastmod: 2026-06-11T16:52:20+02:00
draft: false
description: "Ein technischer Einblick in die Einrichtung eines sicheren Smart-Home-Servers auf einem Fujitsu Futro S9010n unter OpenBSD. Inklusive Nginx-Proxy-Konfiguration, Workarounds für GraalVM-Fehler und automatisierter Thing-Generierung."
license: "MIT"
images: [ "/images/openbsd-home-automation-gateway-with-zigbee2mqtt-and-openhab.png", "edited-image.jpg", "/images/zigbee2mqtt_network.png", "/images/zigbee2mqtt_dashboard.png", "/images/openHAB_addons.png", "/images/openHAB_zigbee_things.png", "/images/openHAB_zigbee_presence_sensor_channels.png", "/images/openHAB-items.png", "/images/openHAB-pages.png", "/images/openHAB-mobile.png" ]

tags: ["OpenBSD", "Zigbee", "Smart Home", "openHAB", "Nginx", "MQTT"]
categories: [ "Smart Home" ]

featuredImage: "/images/openbsd-home-automation-gateway-with-zigbee2mqtt-and-openhab.png"
featuredImagePreview: "/images/openbsd-home-automation-gateway-with-zigbee2mqtt-and-openhab.png"

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
comment:
library:
  css:
  js:
seo:
  images: [ "/images/openbsd-home-automation-gateway-with-zigbee2mqtt-and-openhab.png", "edited-image.jpg", "/images/zigbee2mqtt_network.png", "/images/zigbee2mqtt_dashboard.png", "/images/openHAB_addons.png", "/images/openHAB_zigbee_things.png", "/images/openHAB_zigbee_presence_sensor_channels.png", "/images/openHAB-items.png", "/images/openHAB-pages.png", "/images/openHAB-mobile.png" ]
---

Wer meine bisherigen Beiträge zum Thema Home Automation verfolgt hat, weiß, dass ich mich seit einiger Zeit mit Zigbee beschäftige. Alles begann mit dem Mitschneiden von Frames im Artikel [Zigbee-Sniffing unter OpenBSD: Deep Dive mit dem TI CC2531](/zigbee-sniffing-on-openbsd/). Aus Neugier habe ich mir dann einen neueren Zigbee-Koordinator besorgt und im Folgeartikel [Zigbee Home Automation auf OpenBSD: Sonoff Dongle-M und openHAB](/sonoff-dongle-m-with-openhab-on-openbsd/) über die Anbindung an eine Automatisierungsplattform berichtet. 

Vor Kurzem habe ich einen Posten gebrauchte Thin Clients ergattert, die sich als perfekte Hardware für einen dedizierten Smart-Home-Controller herausgestellt haben. Hier beschreibe ich, wie ich ein stabiles, isoliertes Gateway auf Basis von OpenBSD 7.9-snapshot, Zigbee2MQTT, Mosquitto und openHAB aufgebaut habe.

---

## Der Hardware-Stack

Das Fundament des Setups ist ein [Fujitsu Futro S9010n](https://www.ebay.de/sch/i.html?_nkw=Fujitsu+Futro+S9010n&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=FutroS9010n&toolid=10001&mkevt=1). Diese kleinen Kisten sind hervorragende Heimserver. Für weniger Geld als für ein aktuelles Raspberry-Pi-Kit bekommt man einen vollwertigen, geschlossenen x86-Rechner mit folgender Ausstattung:

* **20 GB RAM** (Ausgeliefert mit 8 GB, aber ich konnte ihn auf 20 GB aufrüsten. Mit 32 GB wurde er allerdings unerträglich langsam)
* **64 GB SSD**
* **Lüfterlose Kühlung** (völlig lautlos, ideal für den Wohnbereich oder den Netzwerkschrank)
* **Zwei native serielle Anschlüsse**

{{< admonition type="warning" title="OpenBSD Grafik-Eigenheiten" open=true >}}
Es gibt eine Besonderheit, wenn man OpenBSD auf dem Futro S9010n betreibt: Der Bildschirm bleibt komplett schwarz, sobald `inteldrm` aktiviert ist. Da das System bei mir ohnehin komplett headless als Server läuft, stört mich das nicht weiter – man sollte es nur bei der Erstinstallation bedenken.
{{< /admonition >}}

Aus Sicherheitsgründen habe ich den Server in ein eigenes IoT-VLAN gesperrt. Die Netzwerkkommunikation mit dem physischen Zigbee-Mesh läuft über das Netzwerk zu einem [SONOFF Zigbee 3.0 USB Dongle Plus (Dongle-M)](https://www.ebay.de/itm/388971238285?mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=DongleM&toolid=10001&mkevt=1). Dadurch sind die ungesicherten Endgeräte komplett von meinem produktiven Heimnetzwerk isoliert.

Als die Basis stand, habe ich eine Reihe von Smart-Home-Geräten eingebunden:
* **Strom:** Ein [4er-Pack SONOFF Zigbee Smart Steckdosen](https://www.ebay.de/itm/385779868557?mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=SonoffSteckdose&toolid=10001&mkevt=1) (praktisch, um das Mesh-Netzwerk als Router zu erweitern).
* **Klima:** Ein [SONOFF Smart Zigbee 3.0 Thermostat](https://www.ebay.de/itm/388766053368?mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Thermostat&toolid=10001&mkevt=1) sowie mehrere [SONOFF SNZB-02P Temperatur- und Feuchtigkeitssensoren](https://www.ebay.de/itm/384895576208?mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=TempFeuchtSensor&toolid=10001&mkevt=1).
* **Präsenz & Beleuchtung:** Der auf Mikrowellen-Radar basierende [SONOFF SNZB-06P Präsenzsensor](https://www.ebay.de/itm/386285050110?mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=PresenceSensor&toolid=10001&mkevt=1), der Standard-[GU10 Smart Zigbee LED Glühbirnen](https://www.ebay.de/sch/i.html?_nkw=Smart+WiFi+Zigbee+LED+Glühbirne+GU10&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Gluehbirne&toolid=10001&mkevt=1) ansteuert, und eine passende [Zigbee Deckenlampe](https://www.ebay.de/itm/326908849090?mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Deckenlampe&toolid=10001&mkevt=1).

---

## Software-Konfiguration

Die Software-Architektur ist klassisch geschichtet: Nginx nimmt den externen Traffic an, übernimmt die Authentifizierung und leitet die Anfragen per Reverse-Proxy an openHAB oder Zigbee2MQTT weiter. Die Dienste kommunizieren untereinander über den Mosquitto MQTT-Broker. Die Verwaltung der Konfiguration erfolgt via OpenVOX, während ein einfaches tägliches Shell-Script die Backups übernimmt.

### 1. Zigbee2MQTT
Die zentrale Zigbee-Übersetzung läuft auf Port `8080`. Die Verbindung zum Adapter wird über TCP aufgebaut, da sich der Sonoff Dongle-M im isolierten IoT-Netzwerksegment befindet.

{{< highlight yaml >}}
version: 5
homeassistant:
  enabled: true
  experimental_event_entities: true
  legacy_action_sensor: true
frontend:
  enabled: true
  host: 127.0.0.1
  base_url: /zigbee2mqtt
  port: 8080
mqtt:
  base_topic: zigbee2mqtt
  server: mqtt://localhost
  version: 5
serial:
  port: tcp://sonoff.example.com:6638
  baudrate: 115200
  adapter: ember
  rtscts: false
advanced:
  log_level: info
  log_directory: /var/log/zigbee2mqtt
  log_file: zigbee2mqtt_%TIMESTAMP%.log
  log_rotation: true
  log_output:
    - file
  network_key:
    # ... secret key ...
{{< /highlight >}}

{{< admonition type="note" title="Konfigurationsmanagement-Hürden" open=true >}}
Diese Datei mit deklarativen Werkzeugen wie OpenVox oder Ansible zu verwalten, ist mühsam. Da der Zigbee2MQTT-Daemon die Datei eigenständig modifiziert (z. B. beim Ändern von Einstellungen oder Anlernen neuer Geräte), kommt es hier schnell zu Konflikten mit dem Konfigurationsmanagement.
{{< /admonition >}}

### 2. Mosquitto & openHAB 5.1.3
Mosquitto macht unter OpenBSD keine Arbeit: Einfach das Paket installieren, per `rcctl` aktivieren und starten.

Die Installation von openHAB verläuft ähnlich unspektakulär. Um Konflikte mit dem Frontend von Zigbee2MQTT zu vermeiden, muss der Standardport in der `/etc/openhab.conf` auf `8081` umgebogen werden:

{{< highlight bash >}}
OPENHAB_HTTP_PORT=8081
{{< /highlight >}}

### 3. Nginx Reverse Proxy
Über Nginx läuft openHAB direkt auf dem Root-Pfad `/`, während das Web-Interface von Zigbee2MQTT unter `/zigbee2mqtt` über eine HTTP-Basic-Auth-Absicherung geschützt wird.

{{< highlight nginx >}}
# MANAGED BY OpenHAB
server {
  listen       *:443 ssl;
  listen       [::]:443 ssl ipv6only=on;

  server_name  ha ha.example.com;

  http2 off;
  ssl_certificate      /etc/puppetlabs/puppet/ssl/certs/ha.example.com.pem;
  ssl_certificate_key  /etc/puppetlabs/puppet/ssl/private_keys/ha.example.com.pem;

  index  index.html index.htm index.php;
  access_log            /var/www/logs/ssl-ha.example.com.access.log;
  error_log             /var/www/logs/ssl-ha.example.com.error.log;

  location /zigbee2mqtt {
    auth_basic           "Home Automation Area";
    auth_basic_user_file /conf/.htpasswd;
    proxy_pass            http://127.0.0.1:8080;
    proxy_read_timeout    90s;
    proxy_send_timeout    90s;
    proxy_http_version    1.1;
    proxy_set_header      Host $host;
    proxy_set_header      X-Real-IP $remote_addr;
    proxy_set_header      X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header      X-Forwarded-Host $host;
    proxy_set_header      X-Forwarded-Proto $scheme;
    proxy_set_header      Proxy "";
    client_body_buffer_size 128k;
    client_max_body_size 8m;
    proxy_buffers 4 32k;
    proxy_redirect http:// https://;
    proxy_set_header Connection "upgrade";
    proxy_set_header Upgrade $http_upgrade;
  }

  location / {
    proxy_pass            http://127.0.0.1:8081;
    proxy_read_timeout    90s;
    proxy_send_timeout    90s;
    proxy_http_version    1.1;
    proxy_set_header      Host $host;
    proxy_set_header      X-Real-IP $remote_addr;
    proxy_set_header      X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header      X-Forwarded-Host $host;
    proxy_set_header      X-Forwarded-Proto $scheme;
    proxy_set_header      Proxy "";
    client_body_buffer_size 128k;
    client_max_body_size 8m;
    proxy_buffers 4 32k;
    proxy_redirect http:// https://;
    proxy_set_header Connection "upgrade";
    proxy_set_header Content-Type $http_content_type;
    proxy_set_header Upgrade $http_upgrade;
  }
}
{{< /highlight >}}

---

## Geräte-Pairing & die OpenBSD-Sackgasse

Nach dem Reload von Nginx funktionierte das Anlernen der Endgeräte im Zigbee2MQTT-Dashboard problemlos: Pairing-Modus in der UI aktivieren, Knopf am Sensor drücken und das Gerät wird interviewt. Die Engine liefert eine gute Übersicht des physischen Mesh-Netzwerks:

{{< figure
    src="/images/zigbee2mqtt_network.png"
    alt="Zigbee network mesh"
    caption="Das visualisierte Zigbee-Mesh-Netzwerk"
    class="ma0 w-75"
>}}

Zusätzlich sieht man direkt die aktuellen Sensorwerte im Dashboard:

{{< figure
    src="/images/zigbee2mqtt_dashboard.png"
    alt="Zigbee devices dashboard"
    caption="Das Zigbee2MQTT Geräte-Dashboard"
    class="ma0 w-75"
>}}

Das automatische Übernehmen dieser erkannten Geräte in openHAB lief dann jedoch gegen die Wand :/

### Die Home Assistant Binding Exception
Normalerweise nutzt man in openHAB das **Home Assistant Binding**, um die von Zigbee2MQTT via MQTT gesendeten Discovery-Topics automatisch einzulesen. Das würde die entsprechenden Things und Channels ohne manuellen Aufwand anlegen. Unter openHAB 5.1 stürzt dieses Binding auf OpenBSD jedoch ab, da es auf native Komponenten via GraalVM setzt, die mit weniger verbreiteten Betriebssystemen Probleme haben.

Ich habe das Problem schließlich im [openHAB-Community-Forum](https://community.openhab.org/t/home-assistant-binding-exception-on-openbsd/169485) geschildert. Der Austausch dort war hervorragend: Ich bekam extrem schnell hilfreiche Antworten und wurde auf das dazugehörige Issue auf [GitHub (#19276)](https://github.com/openhab/openhab-addons/issues/19276) hingewiesen.

{{< admonition type="info" title="Lösung in Sicht" open=true >}}
Die GraalVM-Entwickler integrieren in einem der nächsten Meilensteine ein Konfigurations-Flag, welches den Support für Plattformen abseits des Mainstreams wiederherstellt. Für openHAB 5.2 reicht es zeitlich nicht mehr, aber mit Version 5.3 sollte das Problem offiziell gelöst sein.
{{< /admonition >}}

### Der Workaround: Eine eigene Discovery-Bridge
Da ich keine Lust hatte, Dutzende von Channels händisch in der openHAB-UI anzulegen, habe ich ein kleines Python-Hilfsscript geschrieben, das als Daemon läuft. Es lauscht auf die Konfigurationstopics des Mosquitto-Brokers und schreibt daraus automatisch passende `.things`-Dateien nach `/etc/openhab/things/`.

Das Script ist aktuell noch stark auf meine spezifischen Geräte zugeschnitten, reicht für den Übergang aber völlig aus und spart viel Klickarbeit. Der Code liegt auf GitHub: [github.com/buzzdeee/mqtt_discovery_bridge](https://github.com/buzzdeee/mqtt_discovery_bridge).

### openHAB einrichten: Add-ons und automatische Things

Damit openHAB die über das Script generierten Datenstrukturen auch versteht, müssen zuerst die passenden Erweiterungen installiert werden. 

Im openHAB Add-on Store habe ich dafür das grundlegende **MQTT Binding** sowie die für die Payload-Verarbeitung notwendigen Transformationen **Jinja**, **JSONPath** und **Regex** installiert.

{{< figure
    src="/images/openHAB_addons.png"
    alt="Installierte openHAB Add-ons mit MQTT und Transformationen"
    caption="Die benötigten MQTT- und Transformations-Add-ons in openHAB"
    class="ma0 w-75"
>}}

Nachdem die Add-ons aktiv waren und das Python-Script die Dateien im Konfigurationsverzeichnis abgelegt hatte, wurden die Geräte sofort erkannt. Ein Blick unter **Settings -> Things** zeigt das Ergebnis: Alle Zigbee-Geräte sind ohne manuelles Anlegen online.

{{< figure
    src="/images/openHAB_zigbee_things.png"
    alt="openHAB Things-Liste mit erkannten Zigbee-Geräten"
    caption="Die automatisch über die Konfigurationsdateien erzeugten Things"
    class="ma0 w-75"
>}}

Am Beispiel des SONOFF Präsenzsensors (Mikrowellen-Radar) sieht man die Struktur der übertragenen Daten. Die UI listet alle verfügbaren Channels wie Präsenzerkennung, Helligkeitswerte und Bewegungsstatus sauber auf. Diese können nun direkt mit Items verknüpft werden.

{{< figure
    src="/images/openHAB_zigbee_presence_sensor_channels.png"
    alt="Kanal-Layout des SONOFF Präsenzsensors in openHAB"
    caption="Die erzeugten Channels des Präsenzsensors"
    class="ma0 w-75"
>}}

### Items verknüpfen, Pages bauen und die Mobile-App

Um die Kanäle nutzbar zu machen, legt man in openHAB **Items** an und verknüpft sie mit den entsprechenden Channels der Things. 

Ich habe testweise zwei Items erstellt und sie mit dem Effekt sowie der Lichtsteuerung verbunden.

{{< figure
    src="/images/openHAB-items.png"
    alt="openHAB Items-Konfiguration mit verknüpften Kanälen"
    caption="Verknüpfung der Hardware-Channels mit openHAB Items"
    class="ma0 w-75"
>}}

Nachdem die Items funktionierten, habe ich sie über den integrierten Layout-Manager auf einer neuen Landingpage platziert, um die Bedienung zu testen.

{{< figure
    src="/images/openHAB-pages.png"
    alt="Dashboard-Layout in openHAB UI"
    caption="Das erste einfache Test-Dashboard mit zwei Steuerungselementen"
    class="ma0 w-75"
>}}

Der Vorteil dieses Setups ist die direkte Integration in das openHAB-Ökosystem. Sobald man die offizielle openHAB Mobile-App im lokalen Netzwerk öffnet, wird die neu gebaute Seite ohne zusätzlichen Konfigurationsaufwand synchronisiert. 

Die Steuerung funktioniert direkt vom Smartphone aus und erlaubt es mir, die Lampen im Raum verzögerungsfrei zu schalten.

{{< figure
    src="/images/openHAB-mobile.png"
    alt="openHAB Mobile-App mit Raum-Dashboard"
    caption="Die Dashboard-Ansicht in der mobilen openHAB-App"
    class="ma0 w-75"
>}}

---

## Swiss-Army Debugging-Snippets

Beim Entwickeln der Discovery-Bridge und der Analyse der JSON-Payloads waren neben den normalen Logfiles vor allem diese CLI-Befehle sehr nützlich:

### MQTT-Traffic analysieren
{{< highlight bash >}}
# Gezielte Einstellungsänderungen überwachen
mosquitto_sub -h 127.0.0.1 -v -t "zigbee2mqtt/Spot am 3D Drucker/set"

# Alle Statusmeldungen eines bestimmten Geräts mitlesen
mosquitto_sub -h 127.0.0.1 -v -t "zigbee2mqtt/Spot am 3D Drucker/+"
{{< /highlight >}}

### Test-Payloads manuell senden
{{< highlight bash >}}
mosquitto_pub -h 127.0.0.1 -t "zigbee2mqtt/Spot am 3D Drucker/set" -m '{"effect": "blink"}'
{{< /highlight >}}

### Bereinigung über die openHAB Karaf-Konsole
{{< highlight bash >}}
# Mit der lokalen openHAB-Konsole verbinden
ssh openhab@localhost -p 8101

# Aktive Things auflisten
openhab:things list

# Ein fehlerhaftes Thing manuell löschen
openhab:things remove sony:scalar:fcf1523cd795

# Fehlerhafte oder alte Binding-Bundles aufspüren und deinstallieren
openhab> bundle:list | grep -i sony
# 369 | Active  |  80 | 4.3.0.202412181559    | openHAB Add-ons :: Bundles :: Sony Binding
openhab> bundle:uninstall 369

# Die automatische Inbox aufräumen
openhab:inbox list
openhab:inbox clear
{{< /highlight >}}

---

## Wie geht es weiter?

Das System läuft auf der lüfterlosen Hardware absolut stabil. Die Zeit bis zum Release von openHAB 5.3 (und dem damit erwarteten GraalVM-Fix) kann ich mit meinem Discovery-Script problemlos überbrücken.

Da die Datenübertragung nun zuverlässig funktioniert, stehen als Nächstes die Grundlagen der Strukturierung an. Da ich mich noch in die Basis-Bausteine einarbeite, gibt es noch einiges zu tun:

* **Erstellen und Verknüpfen von Items mit Channels:** Vertiefung bei komplexeren Item-Typen (Dimmer, Farbe, Zahlenwerte) und der korrekten Anwendung von Transformationen, um Rohdaten sauber darzustellen.
* **Gruppieren von Items:** Nutzung von Gruppen-Items, um entweder ganze Räume gleichzeitig zu schalten oder Durchschnittswerte (z. B. Temperaturen) zu berechnen.
* **Aufbau eines sinnvollen Dashboards:** Abseits von reinen Testseiten ein übersichtliches, alltagstaugliches Interface für Smartphone und Desktop entwerfen.
* **Das semantische Heim-Modell:** Abbildung des Hauses über das Semantic Model von openHAB (Locations, Equipment, Points), damit das System versteht, wo welcher Sensor physisch platziert ist.
* **Erste Automatisierungs-Regeln schreiben:** Einfache eventbasierte Scripte aufsetzen – beispielsweise, um das Licht am 3D-Drucker automatisch einzuschalten, wenn der Präsenzsensor Bewegung meldet.
* **Persistence & Daten-Logging:** Einrichtung von rrd4j oder InfluxDB, um Temperatur- und Feuchtigkeitsverläufe langfristig aufzuzeichnen, ohne den Flash-Speicher des Thin Clients unnötig zu belasten.

