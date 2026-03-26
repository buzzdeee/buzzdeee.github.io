---
title: "Eigene Social Icons in Hugo LoveIt nachrüsten"
subtitle: "So integrierst du Ko-fi, Liberapay und andere Plattformen nahtlos in dein Theme"
date: 2026-03-26T16:06:36+01:00
lastmod: 2026-03-26T16:06:36+01:00
draft: false
description: "Lerne, wie du das Hugo LoveIt Theme erweiterst, um Social Media Icons wie Ko-fi oder Liberapay hinzuzufügen, die standardmäßig nicht unterstützt werden."
license: "MIT"
images: []

tags: [ "Hugo", "LoveIt", "Webdesign", "Tutorial", "FontAwesome" ]
categories: [ "Tutorials" ]

featuredImage: "/images/custom_social_links.png"
featuredImagePreview: "/images/custom_social_links.png"

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
  enable: false
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
  images: []
  # ...
---

## Warum das Ganze?

Das Hugo-Theme **LoveIt** bietet eine großartige Out-of-the-Box-Lösung für Social Media Icons. Doch was passiert, wenn man Plattformen nutzt, die nicht im Standard-Repertoire enthalten sind? Wer seine Arbeit über *Ko-fi*, *Liberapay* oder spezialisierte Netzwerke finanziert, möchte diese natürlich prominent in der Sidebar oder im Footer verlinken.

Da LoveIt auf Font Awesome basiert, ist die Lösung eigentlich schon an Bord – wir müssen dem Theme nur beibringen, wie es die neuen Links erkennt und die passenden Icons zuordnet.

## Die Umsetzung

Um ein neues Icon hinzuzufügen, müssen wir nicht im Core-Code des Themes wühlen. Wir nutzen das Prinzip der **Overwrites** in Hugo.

### Die Konfiguration (config.toml)

Zuerst fügst du deinen Link ganz normal unter [params.social] hinzu. Da LoveIt das Icon anhand des Namens sucht, wählen wir einen Bezeichner, der auch bei Font Awesome existiert.

{{< highlight yaml "linenos=table,open=false" >}}
[params.social]
  # Diese Namen müssen mit den Keys in der social.yml übereinstimmen
  kofi = "dein-name"
  liberapay = "dein-name"
{{< /highlight >}}
### Die Umsetzung über `social.yml`

Anstatt tief in die HTML-Templates einzugreifen, nutzt LoveIt eine zentrale Definitionsdatei für alle Social-Media-Dienste. Diese befindet sich standardmäßig unter `assets/data/social.yml`.

#### Die `social.yml` überschreiben

Kopiere die Datei zuerst in dein lokales Projektverzeichnis, damit deine Änderungen bei einem Theme-Update nicht verloren gehen:
`themes/LoveIt/assets/data/social.yml` ➔ `assets/data/social.yml`

#### Neue Dienste registrieren

Öffne die kopierte Datei und füge am Ende (oder an der gewünschten Position) deine neuen Dienste hinzu. LoveIt unterscheidet hierbei zwischen FontAwesome und Simpleicons (SVG).

{{< highlight yaml "linenos=table,open=false" >}}
# 082: Ko-Fi (Nutzt FontAwesome Brands)
kofi:
  Weight: 39
  Title: Ko-Fi
  Prefix: https://ko-fi.com/
  Icon:
    Class: fa-brands fa-ko-fi

# 083: Liberapay (Nutzt die integrierte SVG-Library)
liberapay:
  Weight: 37
  Title: Liberapay
  Prefix: https://liberapay.com/
  Icon:
    Simpleicons: liberapay
{{< /highlight >}}

## 🔍 Wo finde ich die richtigen Icon-Namen?

Damit das Icon korrekt angezeigt wird, musst du den exakten Klassennamen kennen. Je nachdem, welche Quelle du nutzt, gehst du so vor:

1. **Für Font Awesome (Klasse)** Besuche die offizielle [Font Awesome Icon-Suche](https://fontawesome.com/icons).
  * Suche nach deiner Plattform (z. B. "Ko-fi").
  * Achte darauf, den Filter "Free" zu setzen.
  * Kopiere den Klassennamen. In der `social.yml` musst du meist das Präfix für Brands (`fa-brands`) oder Solid-Icons (`fa-solid`) voranstellen.
      * Beispiel: `fa-brands fa-ko-fi`

2. **Für Simple Icons (SVG)** Wenn ein Icon bei Font Awesome fehlt (wie oft bei Open-Source-Projekten), schau in das lokale Verzeichnis deines Themes: `./themes/LoveIt/assets/lib/simple-icons/icons/`
  * Der Name der `.svg` Datei (ohne die Endung) ist dein Key für die `social.yml`.
  * **Wichtig**: Der Name muss exakt übereinstimmen (meist alles kleingeschrieben). Wenn die Datei `liberapay.svg` heißt, trage `liberapay` ein.

### Der Joker: Eigene SVG-Icons einbinden

Was aber, wenn eine ganz neue Plattform oder ein sehr nischiger Dienst weder bei FontAwesome noch in den simple-icons zu finden ist? Kein Problem, du kannst jedes beliebige SVG-Icon selbst hinzufügen.

#### Schritt 1: Das Icon speichern

Besorge dir das Logo als `.svg` Datei (viele Brands bieten diese in ihrem Presse-Kit an). Speichere die Datei in deinem lokalen Projektordner unter: `assets/lib/simple-icons/icons/mein-neuer-dienst.svg`

{{< admonition type="info" open=true >}}
**Wichtig**: Erstelle die Ordnerstruktur in deinem Root-Verzeichnis (`assets/...`), falls sie dort noch nicht existiert. Hugo bevorzugt deine lokalen Dateien gegenüber denen im `themes`-Ordner.
{{< /admonition >}}

{{< admonition type="info" open=true >}}
Wenn du nur ein hochauflösendes PNG oder JPG des Logos hast, kannst du es automatisch vektorisieren lassen.

* **Tools**: Nutze kostenlose Online-Konverter wie [Vector Magic](https://vectormagic.com/) oder [Adobe Express Logo Converter](https://www.adobe.com/express/feature/image/convert/jpg-to-svg).
* **Vorgehen**: Bild hochladen → Vektorisierung starten → Als .svg exportieren.
* **Wichtig**: Achte darauf, dass der Hintergrund transparent ist, sonst hast du später einen weißen Kasten um dein Icon.
{{< /admonition >}}

#### Schritt 2: In der social.yml registrieren

Nun kannst du dein eigenes Icon genau wie ein Standard-Icon ansprechen. Der Name unter `Simpleicons` muss exakt dem Dateinamen entsprechen, den du gerade vergeben hast:

{{< highlight yaml "linenos=table,open=false" >}}
# 084: Mein Neuer Dienst
mein-neuer-dienst:
  Weight: 40
  Title: Mein Dienst
  Prefix: https://beispiel.de/
  Icon:
    Simpleicons: mein-neuer-dienst
{{< /highlight >}}

{{< figure
    src="/images/custom_social_links.png"
    alt="Custom Social Links"
    caption="Custom Social Links"
    class="ma0 w-75"
>}}

## Fazit

Mit Hugo LoveIt bist du nicht auf die Standard-Vorgaben angewiesen. Ob über FontAwesome, die riesige SimpleIcons-Library oder sogar eigene SVG-Dateien – du hast die volle Kontrolle darüber, wie und wo du deine Profile präsentierst. Durch das Spiegeln der social.yml in dein lokales Verzeichnis bleibt dein Setup sauber, sicher vor Updates und jederzeit erweiterbar.

Viel Erfolg beim Netzwerken!
