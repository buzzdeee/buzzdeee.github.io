---
title: "Das Smart Home absichern: Eigene Eltern- & Kinder-Rollen in openHAB implementieren"
subtitle: "Ein praktischer Workaround für maßgeschneiderte Eltern- und Kinder-Ansichten"
date: 2026-06-20T11:30:25+02:00
lastmod: 2026-06-20T11:30:25+02:00
draft: false
description: "Ein Einblick in die Umgehung der nativen Rollenbeschränkungen von openHAB, um maßgeschneiderte 'Eltern'- und 'Kinder'-Zugriffskontrollen für das openHAB UI-Layout und dessen semantischen Modell zu erstellen."
license: "MIT"
images: [ "/images/openhab-users-and-roles.png", /images/openhab-default-list-item-visibility-options.png ]

tags: ["openHAB", "Smart Home", "Security"]
categories: ["Smart Home", "Tutorials"]

featuredImage: "/images/openhab-users-and-roles.png"
featuredImagePreview: "/images/openhab-users-and-roles.png"

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
  images: [ "/images/openhab-users-and-roles.png", /images/openhab-default-list-item-visibility-options.png ]
  # ...
---

Meine Hausautomation läuft vollständig auf OpenBSD, ein solides Fundament für die Verwaltung unseres Haushalts. Doch selbst auf einem sicheren Betriebssystem kann die interne Familienlogistik eine Herausforderung sein – besonders wenn die Kinder entdecken, dass sie den Fernseher im Wohnzimmer von ihren Tablets aus ausschalten oder versehentlich an den Einstellungen des Hauptthermostats herumspielen können. 

Mit dem Ausbau meines Smart Homes stand ich vor einer klaren Notwendigkeit: **Ich musste bestimmte Geräte und sensible Steuerelemente vor meinen Kindern verbergen, während für die Erwachsenen weiterhin alles sichtbar bleiben sollte.**

Das klingt nach einer Standardfunktion für jedes Mehrbenutzersystem. Die Umsetzung in openHAB entpuppte sich jedoch als eine faszinierende Reise durch Backend-Datenbanken, die Einschränkungen der Karaf-Konsole und die unschätzbare Weisheit der Community. Hier ist die Geschichte, wie ich einen Weg um diese strukturellen Einschränkungen herum gefunden habe, um endlich das Setup zu erreichen, nach dem ich gesucht habe – zusammen mit einer Schritt-für-Schritt-Anleitung, damit du das auch tun kannst.

Werfe gerne auch einen Blick auf [/categories/smart-home/](/categories/smart-home/) für meine anderen Artikel rund um [openHAB](https://www.openhab.org/) auf [OpenBSD](https://www.openbsd.org/).

---

## Das Ziel: Kontextabhängige Sichtbarkeit

Das Ziel war unkompliziert. Ich wollte meinen Haushalt in zwei Hauptgruppen aufteilen:
* **Eltern:** Uneingeschränkte Sichtbarkeit aller Dashboards, Gerätekarten (Equipment Cards) und erweiterten Messwerte (wie Batteriezustände).
* **Kinder:** Ein sauberes, vereinfachtes Layout, das nur das zeigt, was sie wirklich brauchen, während sensible Elemente komplett ausgeblendet werden.

---

## Die Hürden & Entdeckungen

Meine ersten Recherchen stießen schnell an Grenzen. In der Core-Architektur von openHAB erkennt das Framework für die Systemsicherheit strikt nur zwei native, funktionale Rollen:

* **`administrator`**: Erhält uneingeschränkten Zugriff auf alles, einschließlich der Systemeinstellungen, Things, Items, Regeln, Entwickler-Tools (Developer Tools) und des Add-on Stores.
* **`user`**: Erhält standardmäßigen Lese-/Schreibzugriff auf Basis-Items, benutzerdefinierte Seiten und die semantischen Tabs (Overview, Locations, Devices, Properties). Die Systemeinstellungen in der Seitenleiste werden komplett ausgeblendet.

Wenn man versucht, einen Benutzer direkt mit einer benutzerdefinierten Rolle anzulegen, wird dies von openHAB ignoriert oder abgelehnt. Zudem ist die Verwaltung eigener Rollen über die Karaf-Konsole nach wie vor ein [offener Verbesserungswunsch (Issue #2453)](https://github.com/openhab/openhab-core/issues/2453). 

Ein Blick auf die Hilfe-Ausgabe der Karaf-Konsole bestätigte diese starren Grenzen:

{{< highlight bash >}}
openhab> openhab:users ?
Usage: openhab:users list - lists all users
Usage: openhab:users add <userId> <password> <role> - adds a new user with the specified role
Usage: openhab:users remove <userId> - removes the given user
Usage: openhab:users changePassword <userId> <newPassword> - changes the password of a user
{{< /highlight >}}

---

## Der Durchbruch: Direktes Tuning der JSONDB

Um die Beschränkung der Karaf-Konsole zu umgehen, musste ich etwas tiefer graben. Nach einigem Suchen nach dem Ort, an dem openHAB seine Benutzerdatenbank versteckt, fiel mir auf: Das Feld `"roles"` ist ein ganz normales Array! Warum also nicht einfach eine weitere Rolle hineinschreiben? Gesagt, getan – und voilà, es funktioniert! 

Damit die Main UI später ordnungsgemäß lädt, muss ein Benutzer lediglich `user` oder `administrator` als fundamentale Basisrolle behalten. Die eigenen Rollen (wie `parent` oder `child`) werden einfach hinzugefügt, fertig.

Hier ist der genaue Ablauf:

1. Verbinde dich per SSH mit deinem openHAB-Host und stoppe den Dienst vollständig, um ein Überschreiben der Daten im laufenden Betrieb zu verhindern:
{{< highlight bash >}}
doas rcctl stop openhab
{{< /highlight >}}

2. Öffne die Benutzer-Datenbankdatei in einem Texteditor:
{{< highlight bash >}}
vi /var/db/openhab/jsondb/users.json
{{< /highlight >}}

3. Suche die entsprechenden Benutzerprofile und passe das `"roles"`-Array an, um deine eigenen Strings neben den Core-Systemrollen zu platzieren:

{{< highlight json >}}
"my_parent_admin_user": {
  "class": "org.openhab.core.auth.ManagedUser",
  "value": {
    "name": "admin",
    "passwordHash": "...",
    "roles": [
      "administrator",
      "parent"
    ]
  }
},
"other_parent_normal_user": {
  "class": "org.openhab.core.auth.ManagedUser",
  "value": {
    "name": "other_parent",
    "passwordHash": "...",
    "roles": [
      "user",
      "parent"
    ]
  }
},
"one_of_the_childs": {
  "class": "org.openhab.core.auth.ManagedUser",
  "value": {
    "name": "child1",
    "passwordHash": "...",
    "roles": [
      "user",
      "child"
    ]
  }
}
{{< /highlight >}}

4. Speichere die Datei und starte openHAB neu:
{{< highlight bash >}}
doas rcctl start openhab
{{< /highlight >}}

Wenn du nun die aktive Benutzerliste über die Karaf-Konsole abfragst, quittiert openHAB das Setup mit der korrekten Erkennung der Mehrfachrollen:

{{< highlight bash >}}
openhab> openhab:users list
admin (administrator, parent)
child1 (child, user)
other_parent (parent, user)
{{< /highlight >}}

---

## Community-Magie: Sichtbarkeit im semantischen Modell anwenden

Nachdem die Backend-Rollen konfiguriert waren, brauchte ich einen sauberen Weg, um sie auf die automatisch generierten semantischen Seiten (Overview, Locations, Equipment) anzuwenden. Ich fragte im openHAB-Forum nach Rat zum Thema [Custom roles and permissions](https://community.openhab.org/t/custom-roles-and-permissions/169573). 

Die Community lieferte das fehlende Puzzleteil: Das Anpassen der **"Default List Item Widget"-Metadaten** direkt bei den einzelnen Items innerhalb des semantischen Modell-Baums.

Beim Erstellen von Layout-Seiten lassen sich Sichtbarkeitsregeln auf zwei elegante Arten anwenden.

### Methode A: Explizite Rollen-Arrays (`visibleTo`)
Die Core-Layout-Engine verarbeitet strukturelle Array-Flags fehlerfrei. Du kannst Standard-UI-Komponenten wie eine Gerätekarte (Equipment Card) explizit vor den Kindern verbergen:

{{< highlight yaml >}}
- component: oh-equipment-card
  config:
    title: Parent's Bedroom TV
    item: MasterTV_Power
    visibleTo:
      - role:administrator
      - role:parent
{{< /highlight >}}

### Methode B: Inline-Ausdrücke (`visible`)
Alternativ kannst du, falls du Ausdrücke innerhalb deiner benutzerdefinierten Grid-Spalten auswerten möchtest, direkt auf die JavaScript-basierte Context-Engine zugreifen:

{{< highlight yaml >}}
- component: oh-label-card
  config:
    action: navigate
    actionPage: page:BatteryStates
    icon: battery
    title: Battery States
    visible: '=user.roles.includes("parent")'
{{< /highlight >}}

{{< admonition type="tip" title="UI-Konfiguration vs. Code-Tab" open=true >}}
Es gibt einen wichtigen praktischen Unterschied, wenn man diese Optionen in der Main UI einrichtet:
* **Methode A (`visibleTo`)** hingegen stößt im visuellen Dropdown-Auswahlfeld der UI an Grenzen, da man dort standardmäßig nur Häkchen für die nativen Rollen `Administrator` oder `User` setzen kann. Um hier deine benutzerdefinierten Rollen wie `parent` oder `child` einzuschleusen, musst du kurz in den **Code**-Tab des Widgets wechseln und sie manuell im YAML-Code ergänzen.
* **Methode B (`visible`)** kann direkt im visuellen Standard-Editor eingegeben werden. Du trägst die Zeichenfolge `=user.roles.includes("parent")` einfach direkt in das Textfeld "Sichtbarkeit" (Visibility) in der Benutzeroberfläche ein.
{{< /admonition >}}

{{< figure
    src="/images/openhab-default-list-item-visibility-options.png"
    alt="Sichtbarkeitsoptionen für Listenelemente"
    caption="Sichtbarkeitsoptionen für Listenelemente"
    class="ma0 w-75"
>}}

Beide Konfigurationen sorgen zuverlässig dafür, dass die entsprechenden Karten für `child1` unsichtbar bleiben, während sie für angemeldete Profile wie `admin` oder `other_parent` voll zugänglich sind.

---

## ⚠️ ENTSCHEIDENDER SCHRITT: Hintertür schließen!

Du kannst die sichersten Layout-Regeln der Welt bauen – sie sind völlig nutzlos, wenn openHAB die Vordertür unverschlossen lässt. Standardmäßig wird openHAB mit einer anonymen Fallback-Option ausgeliefert, um eine nahtlose Browser-Verbindung zu ermöglichen.

Wenn ein Kind das Main-UI-Dashboard öffnet, ohne sich einzuloggen, weist openHAB ihm automatisch eine **implizite Benutzerrolle (Implicit User Role)** zu. Damit umgehen sie sofort deine benutzerdefinierten Sichtbarkeitsregeln und sehen jedes einzelne versteckte Gerät!

{{< admonition type="warning" title="Sicherheitsanforderung" open=true >}}
Du **musst** den anonymen Fallback zwingend deaktivieren, sobald deine Benutzer eingerichtet sind:
1. Öffne die Main UI als Administrator.
2. Navigiere zu **Settings** -> **API Security**.
3. Stelle den Schalter bei **Implicit User Role** auf **OFF**.
{{< /admonition >}}

Nachdem diese Option deaktiviert ist, fordert jedes nicht authentifizierte Gerät strikt zur Anmeldung auf. So wird erzwungen, dass sich die Kinder in ihr eingeschränktes `child`-Profil einloggen müssen, bevor sie überhaupt Oberflächen sehen.

---

## 🛑 Wichtiger Hinweis: Sichtbarkeit ist KEINE absolute Sicherheit

Wie in der offiziellen openHAB-UI-Dokumentation und im obigen Screenshot hervorgehoben wird, **sind Sichtbarkeitsbeschränkungen reiner Komfort für die Benutzeroberfläche und stellen keine echte Sicherheitsbarriere dar.**

Wenn du Eigenschaften wie `visibleTo` oder `visible` nutzt, rendert die openHAB Main UI diese Komponente lediglich nicht mehr im Webbrowser oder in der mobilen App. Im Backend existieren die Items jedoch ganz normal weiter und lauschen auf Befehle. Wenn ein technisch versiertes Kind oder ein Gast weiß, wie man Netzwerkanfragen im Browser analysiert oder direkt mit der openHAB REST API interagiert, kann er die Zustandsänderungen dennoch einsehen und Befehle an die eigentlich versteckte Hardware senden.

Für eine strikte, absolute Hardware-Isolierung musst du diese Sichtbarkeitsfilter mit strengen serverseitigen Ausführungsrechten, Skripten oder dedizierten sekundären Instanzen kombinieren, falls deine Anforderungen an die Netzwerksicherheit absolut sind.

---

## Fazit

Auch wenn openHAB benutzerdefinierte Rollen derzeit noch nicht direkt in den visuellen Administrationsmenüs anbietet: Die Kombination aus mehreren Rollen in der `jsondb` und den standardmäßigen Widget-Sichtbarkeitsausdrücken funktioniert tadellos. Sie schließt die Lücke zwischen mächtigen, ungefilterten Administrationswerkzeugen und einer sicheren, kindgerechten Smart-Home-Oberfläche.

Interessanterweise hat das Aufwerfen dieses Themas im Forum eine faszinierende und weitreichende Debatte innerhalb der Community über rollenbasierte Zugriffskontrolle (RBAC) in Hausautomations-Architekturen ausgelöst. Die richtige Balance zwischen schickem Maskieren von UI-Elementen und harten Sicherheitsgrenzen an der Backend-API ist ein brandheißes Thema, und es ist großartig zu sehen, wie die Community aktiv an zukünftigen Verbesserungen feilt.

Ein riesiges Dankeschön an die openHAB-Community, die mich auf das Metadaten-Mapping für List-Item-Widgets aufmerksam gemacht hat! Hoffentlich hilft dir diese Anleitung dabei, deine eigenen Bedienfelder abzusichern. Viel Spaß beim Automatisieren!

