---
title: "Analytics für deine Hugo-Seite: Der richtige Weg"
subtitle: "Google Analytics, Plausible, Fathom und GoatCounter im Vergleich für das LoveIt Theme"
date: 2026-03-28T09:24:51+01:00
lastmod: 2026-03-28T09:24:51+01:00
draft: false
author: ""
authorLink: ""
description: "Ein umfassender Leitfaden zur Auswahl und Integration von Analytics in eine Hugo-Seite mit dem LoveIt-Theme, inklusive Fokus auf GoatCounter."
license: "MIT"
images: [ "/images/goat-watching-website-stats.png" ]

tags: ["Hugo", "LoveIt", "Analytics", "GoatCounter", "Datenschutz", "Google Analytics"]
categories: ["Tutorials", "Webentwicklung"]

featuredImage: "/images/goat-watching-website-stats.png"
featuredImagePreview: "/images/goat-watching-website-stats.png"

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
share:
  enable: true
comment:
  enable: false
seo:
  images: ["/images/goat-watching-website-stats.png"]
---

## Einleitung

Einen Blog mit Hugo auf GitHub Pages zu betreiben, ist extrem effizient, hat aber eine kleine Schwachstelle: **Sichtbarkeit**. Da GitHub Pages ein statisches Hosting ist, haben wir keinen Zugriff auf Server-Logs. Um zu verstehen, wer unsere Seite besucht, benötigen wir eine clientseitige Lösung.

In diesem Beitrag vergleichen wir die beliebtesten Analytics-Optionen und zeigen, wie man sie im **LoveIt** Theme aktiviert.

## Die Optionen im Vergleich

Bei der Wahl eines Tools geht es meist um die Balance zwischen **Kosten**, **Datenschutz** und **Datentiefe**.

Zum Zeitpunkt der Erstellung dieses Beitrags bietet das LoveIt-Theme integrierte Unterstützung für die folgenden Tools:

1. **[Google Analytics (GA4)](https://analytics.google.com/)**
    * **Der Vibe**: Der Industriestandard. Riesig, mächtig und manchmal etwas erdrückend.
    * **Preis**: Kostenlos.
    * **Vorteile**: Unglaubliche Detailtiefe, kostenlos nutzbar und integriert mit anderen Google-Tools.
    * **Nachteile**: Nicht datenschutzfokussiert, schwerfällig bei der Ladezeit und steile Lernkurve.

2. **[Plausible Analytics](https://plausible.io/)**
    * **Der Vibe**: Die „ethische“ Wahl für Entwickler. Schlank, leichtgewichtig und Open-Source.
    * **Preis**: Kostenpflichtig (ab ~9 €/Monat) oder kostenlos bei Self-Hosting.
    * **Vorteile**: Keine Cookies (DSGVO-konform), extrem schnell, schönes Dashboard.
    * **Nachteile**: Gehostete Version kostet Geld; weniger tiefgehende Analysen als Google.

3. **[Fathom Analytics](https://usefathom.com/)**
    * **Der Vibe**: Datenschutz auf Business-Niveau. Extrem stabil und professionell.
    * **Preis**: Kostenpflichtig (ca. 15 $/Monat).
    * **Vorteile**: Hohe Compliance, umgeht Ad-Blocker via Custom Domains, sehr einfach.
    * **Nachteile**: Teuerster Einstiegspreis für kleine Hobby-Blogs.

4. **[GoatCounter](https://www.goatcounter.com)**
    * **Der Vibe**: Die „Goldlöckchen“-Lösung. Einfache, ehrliche Stats ohne Nutzer-Tracking.
    * **Preis**: Kostenlos für persönliche Nutzung; Bezahlmodelle für Firmen.
    * **Vorteile**: Open-Source, DSGVO-konform, funktioniert sogar ohne JavaScript (Pixel-Tracking).
    * **Nachteile**: Sehr minimalistisches, textbasiertes Interface.

5. **[Yandex Metrica](https://ads.yandex/metrica)**
    * **Der Vibe**: Die kostenlose Alternative mit Heatmaps.
    * **Preis**: Kostenlos.
    * **Vorteile**: Session-Replays (Nutzerverhalten als Video) und Heatmaps gratis enthalten.
    * **Nachteile**: Datenschutzbedenken (Datenverarbeitung in Russland); höchstes Skript-Gewicht.

### Tabellarischer Vergleich

| Feature | Google Analytics 4 | Plausible | Fathom | GoatCounter | Yandex Metrica |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Ideal für** | Power-User & Marketing | Minimalisten & Entwickler | Business & Datenschutz | Persönliche Blogs | Heatmaps & Replays |
| **Preis** | **Kostenlos** | Ab 9 €/Monat | Ab 15 $/Monat | **Kostenlos** (Privat) | **Kostenlos** |
| **Datenschutz** | Niedrig (Cookies) | Hoch (Cookieless) | Hoch (Cookieless) | Hoch (Cookieless) | Niedrig (Cookies) |
| **Einrichtung** | Einfach (Nativ) | Einfach (Nativ) | Einfach (Nativ) | Einfach (Nativ) | Einfach (Nativ) |
| **Skript-Gewicht**| ~30 KB | ~1 KB | ~2 KB | ~3.5 KB | ~45 KB |

{{< admonition type="tip" title="Hinweis zu Ad-Blockern & Genauigkeit" open=true >}}
Nicht alle Analytics-Dienste sind gleich "unsichtbar", wenn es um Blocker geht.
* **Google Analytics (GA4):** Wird am häufigsten blockiert. Fast jeder Ad-Blocker und datenschutzfokussierte Browser (wie Brave oder Firefox) stoppt Googles Skripte standardmäßig. Dir könnten 20–40 % deines tatsächlichen Traffics "fehlen".
* **Plausible & Fathom:** Besser, aber dennoch gelegentlich blockiert, da ihre Domains bekannt sind. Sie bieten jedoch "Custom Domain"- oder "Proxy"-Setups an, wodurch die Skripte wie Teil deiner eigenen Seite wirken und die meisten Blocker umgehen.
* **GoatCounter:** Hat derzeit eine hohe Erfolgsquote, da es leichtgewichtig ist und keine persönlichen Daten trackt. Dennoch bleibt es eine JavaScript-Datei, die blockiert werden *kann*.
* **Die Lösung:** Um 100 % genaue Daten zu erhalten, musst du den Tracker "verstecken". Dies erreichst du durch **Self-Hosting** auf einer eigenen Subdomain (z. B. `stats.deinedomain.de`) oder durch ein **Server-Side**-Tool wie Cloudflare.
{{< /admonition >}}

## Integration der Analytics

Da das **LoveIt**-Theme so hervorragend aufgebaut ist, musst du keinen Code manuell bearbeiten. Wähle einfach deinen Anbieter, besorge dir deine ID bzw. deinen Code und aktualisiere deine `hugo.toml`.

1. **Besorge dir deine Zugangsdaten**
    * **GA4**: Kopiere deine Mess-ID (`G-XXXXXXXXXX`).
    * **Plausible/Fathom**: Verwende deine registrierte Domain oder ID.
    * **GoatCounter**: Nutze deinen „Code“ (die gewählte Subdomain, z. B. `buzzdeee`).
    * **Yandex**: Kopiere deine Counter-ID (z. B. `12345678`).

2. **Konfiguration der `hugo.toml`**

Öffne deine Konfigurationsdatei und trage die Details für den Dienst ein, den du aktivieren möchtest:

{{< highlight toml "linenos=table,open=false" >}}
[params.analytics]
  enable = true
  # Google Analytics 4
  [params.analytics.google]
    id = "G-XXXXXXXXXX"
    # Die „Do Not Track“-Einstellung des Browsers berücksichtigen
    respectDoNotTrack = true
  # GoatCounter
  [params.analytics.goatCounter]
    code = "buzzdeee"
  # Plausible Analytics
  [params.analytics.plausible]
    dataDomain = "deinedomain.de"
  # Fathom Analytics
  [params.analytics.fathom]
    id = ""
    server = "" # Optional: für Self-Hosting
  # Yandex Metrica
  [params.analytics.yandexMetrica]
    id = ""
{{< /highlight >}}

# Meine Top-Empfehlung: GoatCounter 🐐

Für kleine bis mittlere Hugo-Blogs oder Community-Seiten ist **GoatCounter** der klare Sieger. Es trifft genau die goldene Mitte: Es ist so einfach einzurichten wie Google, so privatsphäre-freundlich wie Plausible, aber **kostenlos** für deine persönliche Website. Es ermöglicht dir sogar, deinen Lesern die Seitenaufrufe anzuzeigen, wodurch deine statische Seite lebendiger wirkt.

### Profi-Tipp: Den "Footer-Counter" sichtbar machen

Falls du deinen GoatCounter-Code hinzugefügt hast, aber der Text "X views" im Footer deiner Beiträge nicht erscheint, hast du wahrscheinlich eine kleine, aber entscheidende Einstellung übersehen.

#### 1. Analytics in der `hugo.toml` aktivieren

Selbst wenn du den GoatCounter-Code hinterlegst, wird das LoveIt-Theme das Skript nicht ausführen, solange der globale Analytics-Parameter nicht auf `true` gesetzt ist. Stelle sicher, dass deine Konfiguration so aussieht:

{{< highlight toml "linenos=table,open=false" >}}
[params.analytics]
  enable = true  # <--- DIES IST DER "HAUPTSCHALTER"
  [params.analytics.goatCounter]
    code = "deine-subdomain" # z. B. "buzzdeee"
{{< /highlight >}}

#### 2. Den Schalter in den GoatCounter-Einstellungen umlegen

Standardmäßig schützt GoatCounter deine Daten und erlaubt es deiner Website nicht, deine Statistiken einfach "auszulesen", um sie anzuzeigen. Du musst hierfür explizit die Erlaubnis erteilen:

* Logge dich in dein **GoatCounter-Dashboard** ein.
* Gehe zu **Settings** → **Data collection**.
* Aktiviere das Kontrollkästchen für: "**Allow adding visitor counts on your website**".
* Scrolle nach unten und klicke auf **Save**.

#### Aktualisieren und Überprüfen

Sobald beides eingestellt ist, wird das integrierte Skript des Themes die Daten automatisch abrufen und in den Footer deiner Beiträge einfügen. Falls es immer noch nicht erscheint, überprüfe deine Browser-Konsole (`F12`) – es könnte sein, dass ein Ad-Blocker das Skript während deines Tests blockiert!

# Meine Top-Empfehlung: GoatCounter 🐐

Für kleine bis mittlere Hugo-Blogs oder Community-Seiten ist **GoatCounter** der klare Sieger. Es trifft genau die goldene Mitte: Es ist so einfach einzurichten wie Google, so privatsphäre-freundlich wie Plausible, aber **kostenlos** für deine persönliche Website. Es ermöglicht dir sogar, deinen Lesern die Seitenaufrufe anzuzeigen, wodurch deine statische Seite lebendiger wirkt.

### Profi-Tipp: Den "Footer-Counter" sichtbar machen

Falls du deinen GoatCounter-Code hinzugefügt hast, aber der Text "X views" im Footer deiner Beiträge nicht erscheint, hast du wahrscheinlich eine kleine, aber entscheidende Einstellung übersehen.

#### 1. Analytics in der `hugo.toml` aktivieren

Selbst wenn du den GoatCounter-Code hinterlegst, wird das LoveIt-Theme das Skript nicht ausführen, solange der globale Analytics-Parameter nicht auf `true` gesetzt ist. Stelle sicher, dass deine Konfiguration so aussieht:

{{< highlight toml "linenos=table,open=false" >}}
[params.analytics]
  enable = true  # <--- DIES IST DER "HAUPTSCHALTER"
  [params.analytics.goatCounter]
    code = "deine-subdomain" # z. B. "buzzdeee"
{{< /highlight >}}

#### 2. Den Schalter in den GoatCounter-Einstellungen umlegen

Standardmäßig schützt GoatCounter deine Daten und erlaubt es deiner Website nicht, deine Statistiken einfach "auszulesen", um sie anzuzeigen. Du musst hierfür explizit die Erlaubnis erteilen:

* Logge dich in dein **GoatCounter-Dashboard** ein.
* Gehe zu **Settings** → **Data collection**.
* Aktiviere das Kontrollkästchen für: "**Allow adding visitor counts on your website**".
* Scrolle nach unten und klicke auf **Save**.

#### Aktualisieren und Überprüfen

Sobald beides eingestellt ist, wird das integrierte Skript des Themes die Daten automatisch abrufen und in den Footer deiner Beiträge einfügen. Falls es immer noch nicht erscheint, überprüfe deine Browser-Konsole (`F12`) – es könnte sein, dass ein Ad-Blocker das Skript während deines Tests blockiert!

## Fazit: Welches Tool solltest du wählen?

Die Wahl des richtigen Analytics-Tools für deine Hugo-Seite hängt ganz von deinen Zielen für das Jahr 2026 ab:

* **Die „Goldlöckchen“-Wahl (GoatCounter):** Perfekt für persönliche Blogs. Es ist bei angemessener Nutzung kostenlos, respektiert die Privatsphäre und der integrierte „Live-View“-Zähler im Footer des LoveIt-Themes ist ein fantastischer Bonus.
* **Die professionelle Datenschutz-Wahl (Plausible oder Fathom):** Wenn du ein Unternehmen oder eine Seite mit hohem Traffic betreibst und eine „Sorgenfrei“-Lösung mit einem modernen Dashboard suchst. Beide bieten eine **einmonatige kostenlose Testphase** an, sodass du sie risikofrei testen kannst.
* **Die Wahl für Power-User (Google Analytics 4):** Am besten geeignet, wenn du eine tiefe Integration mit Google Ads oder komplexes Funnel-Tracking benötigst und dich die steile Lernkurve (oder die Cookie-Banner) nicht stören.
* **Die Wahl für UX-Designer (Yandex Metrica):** Wenn du spezifisch **Heatmaps** sehen und Aufzeichnungen darüber sehen möchtest, wie Nutzer kostenlos mit deiner Seite interagieren.

### Warum ich GoatCounter empfehle 🐐

Für einen Hugo-Blog ist **GoatCounter** mein Favorit. Es trifft genau die goldene Mitte: Die Einrichtung ist so einfach wie bei Google, es ist so privatsphäre-freundlich wie Plausible und **kostenlos** für deine persönliche Website. Es erlaubt dir sogar, deinen Lesern die Seitenaufrufe anzuzeigen, wodurch deine statische Seite lebendiger wirkt.

**Unterstütze den Entwickler:**
GoatCounter ist ein unabhängiges Open-Source-Projekt. Wenn du es für deinen Blog nützlich findest, ziehe in Erwägung, den Entwickler über **[GitHub Sponsors](https://github.com/sponsors/arp242)** zu unterstützen. Es hilft dabei, den Dienst für private Nutzer kostenlos zu halten und stellt sicher, dass das Projekt unabhängig bleibt!

## Checkliste für den Erfolg

Bevor du `git push` ausführst, stelle sicher, dass du diese drei kritischen Punkte abgehakt hast:

1. **Der Hauptschalter:** Vergewissere dich, dass `enable = true` unter `[params.analytics]` in deiner `hugo.toml` gesetzt ist. Ohne dies wird LoveIt keines deiner Tracking-Skripte laden!
2. **GoatCounter-API:** Wenn du dich für GoatCounter entscheidest, denke daran, dich in dein Dashboard einzuloggen und das Häkchen bei **„Allow adding visitor counts on your website“** in den Einstellungen zu setzen, damit dein Footer-Zähler die Daten tatsächlich abrufen kann.
3. **Datenschutz beachten:** Wenn du dich für ein Cookie-basiertes Tool wie Google oder Yandex entscheidest, solltest du eine einfache Datenschutzseite hinzufügen, um mit den globalen Vorschriften konform zu bleiben.

**Nächste Schritte:** Wähle deinen Anbieter, schnapp dir deine ID und lege den `enable`-Schalter auf `true`. Dein statischer Blog ist nicht mehr nur eine Sammlung von Dateien – er ist eine lebendige Seite mit einem messbaren Publikum!

