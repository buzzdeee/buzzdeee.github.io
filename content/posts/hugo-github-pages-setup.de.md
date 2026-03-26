---
title: "Modernes Blogging: Hugo, GitHub Pages und eine eigene Domain"
date: 2026-03-23
description: "Ein Leitfaden für IT-Enthusiasten: Von der lokalen Installation unter OpenBSD bis zum automatisierten Deployment via GitHub Actions."
tags: ["Hugo", "LoveIt", "GitHub Pages", "OpenBSD", "DevOps", "Tutorial"]
categories: ["Tutorials"]
featuredImage: "/images/hugo-github-setup.png"
---

Du hast eine Projektidee und suchst nach einem einfachen Weg, diese auf einer eigenen Website zu präsentieren? Mit **Hugo** und **GitHub Pages** ist das im Handumdrehen erledigt. Am Beispiel dieser Seite zeige ich dir, wie unkompliziert der Weg zur eigenen Präsenz ist.

## Voraussetzungen

Die Hürden für dein Projekt „Eigene Website“ sind minimal. Alles, was du benötigst, ist:

* **GitHub Account**: Hier wird dein Code sicher verwaltet und veröffentlicht.
* **Eigene Domain (optional)**: Falls du deine Seite unter einer persönlichen Adresse erreichbar machen möchtest.
* **Betriebssystem mit Terminal**: Eine solide Kommandozeile ist Pflicht (in dieser Anleitung nutzen wir beispielhaft **OpenBSD**).

## Warum Hugo?

Für jemanden, der viel auf der Kommandozeile arbeitet (und als OpenBSD Maintainer minimalistische Ansätze schätzt), ist Hugo ein Segen. Es generiert statisches HTML, was bedeutet:

* **Sicherheit:** Keine SQL-Injections oder PHP-Lücken.
* **Speed:** Die Seite lädt fast instantan.
* **Portabilität:** Der gesamte Content liegt in Markdown-Dateien.

## Der Workflow unter OpenBSD

Die Installation von Hugo, Git, und einigen weiteren hilfreichen Tools ist dank der vorhandenen Packages denkbar einfach:
{{< highlight shell "linenos=table,open=false" >}}
doas pkg_add hugo git nvi ImageMagick
{{< /highlight >}}

## Theme Auswahl

Bevor es richtig losgeht, hast du die Qual der Wahl: Für Hugo steht eine riesige Auswahl an Themes bereit. In der [offiziellen Übersicht](https://themes.gohugo.io/) kannst du die verschiedenen Designs erkunden und direkt in einer Live-Demo testen.

In dieser Anleitung verwende ich als Beispiel das [LoveIt-Theme](https://themes.gohugo.io/themes/loveit/), welches auch das Fundament für die Seite bildet, die du gerade vor dir siehst.

{{< admonition type="info" open=true >}}
**Gut zu wissen**: Dank Hugos strikter Trennung von Inhalten (Content) und Design kannst du das Theme prinzipiell jederzeit wechseln. Beachte jedoch, dass jedes Theme eigene Variablen in der Konfigurationsdatei nutzt. Ein späterer Wechsel kann daher ein wenig Nacharbeit erfordern, um die Einstellungen an das neue Theme anzupassen.
{{< /admonition >}}


## Vorbereitungen

### DNS Konfiguration (optional)

Detaillierte Informationen zur Einbindung von GitHub Pages in deine eigene Domain findest du in der [GitHub-Dokumentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site).

Für eine persönliche Portfolio-Seite ist die Erstellung einer User-Page der einfachste Weg. Gehe dazu wie folgt vor:

1. **Repository erstellen**: Benenne das Repo nach dem Schema `[Dein-GitHub-Username].github.io`.

2. **DNS-Konfiguration**: Um die Seite später über deine eigene Domain erreichbar zu machen (z. B. `buzzdeee.reitenba.ch`), musst du einen CNAME-Eintrag in deinen DNS-Einstellungen setzen, der auf `buzzdeee.github.io` verweist.

{{< admonition type="info" open=true >}}
Falls du (noch) keine eigene Domain hast, wird deine Seite dann unter `[Dein-GitHub-Username].github.io` erreichbar sein.
{{< /admonition >}}

### Git(-Hub) Einstellung

Nimm in GitHub folgende Einstellungen vor, um deine Seite zu veröffentlichen:

1. **Repository erstellen**: Falls noch nicht geschehen, erstelle ein neues Repository. Benutze dabei das Schema [Dein-GitHub-Username].github.io.

2. **GitHub Pages konfigurieren**: Navigiere in den Repository-Einstellungen zum Menüpunkt Pages und trage dort folgende Details ein:
    * **Build and deployment**: Wähle unter *Source* die Option **GitHub Actions** aus.
    * **Custom Domain**: Wenn du deine eigene Domain nutzen möchtest, trage hier deinen Hostnamen (z. B. `buzzdeee.reitenba.ch`) ein und bestätige mit **Save**. Der DNS-Check sollte nach wenigen Augenblicken erfolgreich abgeschlossen sein.
    * **Enforce HTTPS**: Aktiviere diese Checkbox, um sicherzustellen, dass deine Seite verschlüsselt und ausschließlich über HTTPS erreichbar ist.

{{< admonition type="info" open=true >}}
**Hinweis**: Die `Enforce HTTPS` Checkbox kann erst nach dem ersten Deployment aktiviert werden!
{{< /admonition >}}

Deine Einstellungen sollten anschließend wie in diesem Beispiel aussehen:

{{< figure
    src="/images/GitHub_Pages_Settings.png"
    alt="GitHub Pages Konfigurationseinstellungen"
    caption="Beispielhafte Konfiguration der GitHub Pages Settings"
    class="ma0 w-75"
>}}

## Minimales Setup und Initialisierung

Sobald du dich für ein Theme entschieden hast, legen wir das Fundament für dein Projekt.

Zuerst erstellst du ein lokales Verzeichnis, wechselst hinein, initialisierst ein neues Git-Repository und fügst das **LoveIt-Theme** als Submodule hinzu. Das hat den Vorteil, dass du das Theme später kinderleicht auf dem neuesten Stand halten kannst, ohne deinen eigenen Code zu vermischen.


{{< admonition type="info" open=false >}}
**Verzeichnis Erstellen und betreten**

```
mkdir buzzdeee.github.io
cd buzzdeee.github.io
```

**Git Repository initialisieren (mit Hauptzweig 'main')**

```
git init -b main
```

**Theme als Submodule hinzufügen und initialisieren**

```
git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
git submodule update --init --recursive
```
{{< /admonition >}}

## Konfiguration anpassen

Um nicht bei Null anfangen zu müssen, kopieren wir die Beispiel-Konfiguration des Themes in unser Hauptverzeichnis und passen sie mit einem Editor (z. B. `nvi` oder `vi`) an:

{{< highlight shell "linenos=table,open=true" >}}
cp themes/LoveIt/exampleSite/hugo.toml .
nvi hugo.toml
{{< /highlight >}}

In der Datei `hugo.toml` solltest du vor allem folgende Punkte prüfen und individualisieren:

* **BaseURL**: Die finale Adresse, unter der deine Seite erreichbar sein wird (z. B. https://buzzdeee.reitenba.ch/).
* **Autor-Informationen**: Name, Social-Links und Kurzbeschreibung.
* **Spracheinstellungen**: Standard-Sprache sowie die Menüeinträge für die Navigation.
* **Kommentarfunktion**: Für den Start empfiehlt es sich, diese zu deaktivieren (eine detaillierte Einrichtung würde hier den Rahmen sprengen).

Im Folgenden findest du die wichtigsten Konfigurationsbeispiele, die du als Orientierung für deine eigene hugo.toml nutzen kannst:

{{< highlight toml "linenos=table,open=false" >}}
baseURL = "https://buzzdeee.reitenba.ch"

# theme
theme = "LoveIt"
# themes directory
themesDir = "themes"

defaultContentLanguage = "de"

[languages]
  [languages.de]
    weight = 1
    languageCode = "de"
    languageName = "Deutsch"
    title = "Mein Blog"
    [languages.de.menu]
      [[languages.de.menu.main]]
        name = "Posts"
        url = "posts"
        weight = 1
      [[languages.de.menu.main]]
        weight = 2
        identifier = "tags"
        pre = ""
        post = ""
        name = "Tags"
        url = "/tags/"
        title = ""
      [[languages.de.menu.main]]
        weight = 3
        identifier = "categories"
        pre = ""
        post = ""
        name = "Kategorien"
        url = "/categories/"
        title = ""
      [[languages.de.menu.main]]
        weight = 4
        identifier = "documentation"
        pre = ""
        post = ""
        name = "Docs"
        url = "/categories/documentation/"
        title = ""
      [[languages.de.menu.main]]
        weight = 5
        identifier = "about"
        pre = ""
        post = ""
        name = "About"
        url = "/about/"
        title = ""
      [[languages.de.menu.main]]
        weight = 6
        identifier = "github"
        pre = "<i class='fab fa-github' aria-hidden='true'></i>"
        post = ""
        name = ""
        url = "https://github.com/myaccount"
        title = "GitHub"
  [languages.en]
    weight = 2
    languageCode = "en"
    languageName = "English"
    title = "My Blog"
    [languages.en.menu]
      [[languages.en.menu.main]]
        name = "Posts"
        url = "posts"
        weight = 1
      [[languages.en.menu.main]]
        weight = 2
        identifier = "tags"
        pre = ""
        post = ""
        name = "Tags"
        url = "/tags/"
        title = ""
      [[languages.en.menu.main]]
        weight = 3
        identifier = "categories"
        pre = ""
        post = ""
        name = "Categories"
        url = "/categories/"
        title = ""
      [[languages.en.menu.main]]
        weight = 4
        identifier = "documentation"
        pre = ""
        post = ""
        name = "Docs"
        url = "/categories/documentation/"
        title = ""
      [[languages.en.menu.main]]
        weight = 5
        identifier = "about"
        pre = ""
        post = ""
        name = "About"
        url = "/about/"
        title = ""
      [[languages.en.menu.main]]
        weight = 6
        identifier = "github"
        pre = "<i class='fab fa-github' aria-hidden='true'></i>"
        post = ""
        name = ""
        url = "https://github.com/buzzdeee"
        title = "GitHub"
...
  # Author config 
  [params.author]
    name = "buzzdeee"
    email = ""
    link = ""
...
[params.page.comment]
    enable = false
...
{{< /highlight >}}

{{< admonition type="tip" title="Support für Mehrsprachigkeit" open=true >}}
Deine Artikel speicherst du als Markdown-Dateien im Verzeichnis `./content/posts/`. Damit Hugo erkennt, dass zwei Dateien denselben Inhalt in verschiedenen Sprachen darstellen, müssen sie den identischen Dateinamen tragen und sich nur durch das Sprachkürzel unterscheiden.

Beispiel:

* `mein-artikel.de.md` (Deutsche Version)

* `mein-artikel.en.md` (Englische Version)

Nur so kann der Besucher deiner Seite später direkt zwischen den Sprachen des jeweiligen Artikels umschalten.
{{< /admonition >}}


## Der erste lokale Testlauf

Sobald die Konfiguration steht, ist es Zeit für den ersten Probelauf. Starte den Hugo-Server mit dem Befehl `hugo server -D` (das `-D` sorgt dafür, dass auch Entwürfe angezeigt werden):

{{< admonition type="error" title="Fehlermeldung beim Start" open=false >}}
```
ERROR error building site: failed to acquire a build lock: 
open /home/user/GIT/buzzdeee.github.io/.hugo_build.lock: too many open files
```
{{< /admonition >}}

Autsch! Unter OpenBSD stößt man hier oft auf ein klassisches Limit-Problem (`too many open files`). Das liegt daran, dass Hugo standardmäßig sehr viele Dateien gleichzeitig überwacht.

Dafür gibt es zwei Lösungswege:

1. **Die saubere Lösung (System-Limits erhöhen)**: Erhöhe die Werte für `openfiles-max` und `openfiles-cur` in der Datei `/etc/login.conf` auf `4096`. Nach einem erneuten Login sollte `hugo server -D` inklusive der automatischen Dateiüberwachung (Watch-Mode) problemlos starten.
2. **Der "Quick Fix" (Ohne Dateiüberwachung)**: Falls du die Systemkonfiguration nicht ändern möchtest, kannst du Hugo auch mit reduzierten Funktionen starten. Dabei wird auf das automatische Neuladen bei Änderungen verzichtet:

{{< highlight shell "linenos=table" >}}
hugo server --disableFastRender --ignoreCache --noHTTPCache --watch=false
{{< /highlight >}} 

Sobald der Server erfolgreich läuft, kannst du dein Zwischenergebnis im Browser unter http://localhost:1313/ in Echtzeit begutachten und testen.

## Das Avatar-Bild

Für das Profilbild auf deiner Startseite hast du zwei Möglichkeiten:

* **Gravatar**: Ein globaler Dienst, der dein Bild verknüpft mit deiner E-Mail-Adresse bereitstellt.
* **Lokales Bild**: Ein Bild, das direkt in deinem Repository gespeichert ist.

### Vor- und Nachteile von Gravatar

* **Vorteil*: Du musst dein Bild nur an einer zentralen Stelle ([gravatar.com](https://gravatar.com)) aktualisieren, und es wird automatisch auf allen verknüpften Plattformen (GitHub, StackOverflow, dein Blog etc.) angepasst.

* **Nachteil (Datenschutz)**: Um das Bild zu laden, sendet der Browser des Besuchers eine Anfrage an die Gravatar-Server. Dabei wird ein Hash deiner E-Mail-Adresse sowie die IP-Adresse des Besuchers übertragen, was aus Datenschutzsicht (DSGVO) sensibel sein kann.

### Avatar-Konfiguration in der `hugo.toml`

Je nachdem, für welchen Weg du dich entscheidest, hinterlegst du entweder deine E-Mail-Adresse bei `gravatarEmail` oder passt den Pfad bei `avatarURL` an:

{{< admonition type="info" title="Speicherort für Bilder" open=true >}}
```
    [params.home.profile]
      enable = true
```

**Gravatar-E-Mail für das Profilbild (optional)**

```
      gravatarEmail = ""
```

**Lokaler Pfad zum Profilbild**

```
avatarURL = "/images/avatar.png"
```
{{< /admonition>}}

{{< admonition type="tip" title="Speicherort für Bilder" open=true >}}
Statischer Content wie Bilder gehört in das Verzeichnis `./static/`. Das Avatar-Bild sollte also unter `./static/images/avatar.png` abgelegt werden. Hugo macht es beim Build-Prozess dann unter `/images/avatar.png` verfügbar.
{{< /admonition >}}

## Favicon

Damit deine Seite auch im Browser-Tab gut aussieht, benötigst du ein Favicon. Die Datei `favicon.ico` gehört direkt in das Verzeichnis `./static/`.

{{< admonition type="tip" title="Avatar in Favicon konvertieren" open=true >}}
Mit dem Tool `convert` aus dem ImageMagick-Paket kannst du dein Avatar-Bild ganz einfach in ein passendes Favicon umwandeln:
```
convert static/images/avatar.png -define icon:auto-resize=64,48,32,16 static/favicon.ico
```
{{< /admonition >}}

## Die "About"-Seite erstellen

Damit der Menüpunkt „About“ nicht ins Leere führt, müssen wir die entsprechende Inhaltsdatei anlegen. Da Hugo zwischen normalen Blog-Posts und statischen Seiten unterscheidet, platzieren wir diese Datei direkt im `content`-Ordner.

### Erstellung der Dateien

Erstelle für jede Sprache eine eigene Markdown-Datei unter `./content/about/`:

* **Deutsch**: `./content/about/index.de.md`
* **Englisch**: `./content/about/index.en.md`

### Beispiel für den Inhalt (`about/index.de.md`)

{{< highlight markdown "linenos=table,open=false" >}}
---
title: "Über mich"
layout: "single"
nodateline: true
noauthor: true
---

Hier kannst du dich kurz vorstellen, deine Projekte beschreiben oder erzählen, worum es auf diesem Blog geht. Da diese Datei `index.de.md` heißt, wird sie automatisch unter der URL `/about/` (bzw. `/de/about/`) angezeigt.
{{< /highlight >}}

{{< admonition type="tip" title="Zweisprachige Verknüpfung" open=true >}}
Genau wie bei den Blog-Posts erkennt Hugo die Zusammengehörigkeit der Seiten am identischen Pfad/Dateinamen. Wenn ein Besucher auf der „Über mich“-Seite ist und die Sprache wechselt, landet er direkt auf dem englischen Gegenstück `about/index.en.md`.
{{< /admonition >}}

## Deinen ersten Post erstellen

Nachdem das Grundgerüst steht, wird es Zeit für den ersten echten Inhalt. Um einen neuen Blog-Post zu erstellen, nutzt du am besten den `hugo new`-Befehl. Dieser legt die Datei direkt mit dem passenden Grundgerüst (dem sogenannten Frontmatter) an.

### Schritt 1: Dateien anlegen

Erstelle im Terminal nacheinander die deutsche und die englische Version deines Beitrags:

{{< highlight shell "linenos=table" >}}
hugo new posts/mein-erster-beitrag.de.md
hugo new posts/mein-erster-beitrag.en.md
{{< /highlight >}}

### Schritt 2: Den Status auf "Veröffentlicht" setzen

Standardmäßig markiert Hugo neue Beiträge als Entwurf. Damit sie auf deiner Live-Seite erscheinen, musst du im Kopf der Datei (Frontmatter) die Zeile `draft = true` entweder löschen oder auf `false` setzen:

{{< highlight markdown "linenos=table" >}}
title: "Mein erster Beitrag"
date: 2026-03-25T10:00:00+01:00
draft: false
{{< /highlight >}}

{{< admonition type="tip" title="Draft-Modus beim Testen" open=true >}}
Solange `draft = true` gesetzt ist, siehst du den Post nur lokal, wenn du den Server mit der Flag `-D` startest (`hugo server -D`). Auf der fertigen Website auf GitHub werden Entwürfe ignoriert.
{{< /admonition >}}

## Automatisierung mit GitHub Actions

Damit deine Seite bei jedem `git push` automatisch gebaut und veröffentlicht wird, nutzen wir *GitHub Actions*. Erstelle dazu die Datei `.github/workflows/hugo.yaml` in deinem Repository und füge den folgenden Inhalt ein:

{{< highlight yaml "linenos=table,open=false" >}}
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive # to include the theme
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true # LoveIt theme requires extended version (SCSS)

      - name: Build with Hugo
        run: hugo --minify

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
{{< /highlight >}} 

### Aktivierung in den Repository-Settings

Damit GitHub diesen Workflow auch tatsächlich nutzt, falls noch nicht geschehen, musst du eine kleine Änderung in den Einstellungen vornehmen:
Navigiere zu **Settings** > **Pages**. Wähle unter Build and deployment im Dropdown-Menü *Source* die Option **GitHub Actions** aus.

## Sauber bleiben: Die .gitignore Datei

Beim lokalen Testen und Bauen deiner Seite entstehen temporäre Dateien und Verzeichnisse (wie der `public/`-Ordner), die nicht in dein Git-Repository gehören. Um diese auszuschließen, erstelle eine Datei namens `.gitignore` im Hauptverzeichnis:

{{< admonition type="info" title=".gitignore" open=true >}}
**Lokales Build-Verzeichnis (wird auf GitHub neu generiert)**

```
public/
```

**Ressourcen-Cache und temporäre Dateien**

```
resources/
.hugo_build.lock
```
{{< /admonition >}}

## Der finale Git-Push

Jetzt ist alles vorbereitet, um deine Seite live zu schalten. Mit den folgenden Befehlen überträgst du deine lokale Konfiguration in dein GitHub-Repository:

{{< highlight shell "linenos=table" >}}
git add .
git commit -m "Initialer Commit der Hugo-Seite"
git remote add origin git@github.com:buzzdeee/buzzdeee.github.io.git
git push -u origin main
{{< /highlight >}}

### Mögliche Hürde: GitHub Push Protection

Es kann vorkommen, dass GitHub den Push mit einer Fehlermeldung ablehnt. Das liegt meist daran, dass das LoveIt-Theme in der Beispiel-Konfiguration einen Standard-Key für **Mapbox** enthält, den GitHub aus Sicherheitsgründen blockiert:

{{< admonition type="error" title="Fehler: git push rejected" open=false >}}
```
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote: - GITHUB PUSH PROTECTION
remote: Push cannot contain secrets
remote: Mapbox Secret Access Token found in hugo.toml:472
```
{{< /admonition >}}

**Die Lösung**: In diesem speziellen Fall handelt es sich um einen bekannten Demo-Key des Themes. Du hast zwei Möglichkeiten:

1. Lösche den Key in der `hugo.toml` (Zeile 472), falls du Mapbox nicht nutzt.

2. Folge dem Link in der Fehlermeldung (`https://github.com/.../unblock-secret/...`), um GitHub mitzuteilen, dass dieser spezifische Key ignoriert werden soll.

Sobald der Key freigegeben oder entfernt wurde, läuft der git push sauber durch und der Build-Prozess startet automatisch.

{{< admonition type="tip" title="Pro-Tipp: Sicherheit geht vor" open=true >}}
Solltest du später eigene, echte API-Keys nutzen, schreibe diese **niemals** direkt in die hugo.toml. Nutze stattdessen Umgebungsvariablen oder die **GitHub Actions Secrets**. So bleiben deine Zugangsdaten geheim und landen nicht im öffentlichen Code deines Repositories.
{{< /admonition >}}

## Fazit & Checkliste

Herzlichen Glückwunsch! Wenn alles geklappt hat, ist deine persönliche Website nun weltweit unter deiner eigenen Domain erreichbar. Dank Hugo und GitHub Actions hast du jetzt ein extrem schnelles, sicheres und wartungsarmes System.

Bevor du dich an deinen nächsten Artikel setzt, hier eine kurze **Erfolgs-Checkliste**:

* [ ] **Domain erreichbar?** Prüfe, ob deine URL (z. B. `https://buzzdeee.reitenba.ch`) korrekt auflöst.

* [ ] **HTTPS aktiv?** Erscheint das Schloss-Symbol in der Adresszeile deines Browsers?

* [ ] **Sprachwechsel ok?** Funktioniert das Umschalten zwischen Deutsch und Englisch auf der "About"-Seite und bei deinen Posts?

* [ ] **Favicon sichtbar?** Wird dein individuelles Icon im Browser-Tab angezeigt?

* [ ] **GitHub Action grün?** Schau in deinem GitHub-Repo unter dem Reiter Actions, ob der Build ohne Fehler durchgelaufen ist.

### Was kommt als Nächstes?

Du kannst nun jederzeit neue Inhalte erstellen, indem du einfach eine neue Markdown-Datei anlegst und per `git push` hochlädst. Den Rest erledigt die Automatisierung für dich.

Beachte bitte, dass dieser Guide nur das Grundgerüst behandelt. Das LoveIt-Theme bietet noch weitaus mehr mächtige Funktionen, auf die ich hier nicht im Detail eingegangen bin, um den Rahmen nicht zu sprengen. Dazu gehören unter anderem:

* **Mapbox-Integration**: Für interaktive Karten.

* **Kommentarsysteme**: Einbindung von Disqus, Utterances oder Giscus.

* **Social Media**: Verknüpfung deiner Profile in der Navigation oder im Fußbereich.

* **Shortcodes**: Spezielle Hugo-Befehle für Galerien, Musik-Player oder YouTube-Videos.

Die weitere Entdeckungsreise durch die [LoveIt-Dokumentation](https://hugoloveit.com/) und das Ausprobieren neuer Features liegt nun ganz bei dir.

**Viel Erfolg und Spaß mit deinem neuen Blog!**
