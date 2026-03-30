---
title: "Hugo Themes anpassen: Einen mehrsprachigen Custom-Footer hinzufügen"
subtitle: "Wie du Theme-Layouts sicher überschreibst und bestehende Social-Media-Konfigurationen im LoveIt-Theme wiederverwendest."
date: 2026-03-30T19:59:11+02:00
lastmod: 2026-03-30T19:59:11+02:00
draft: true
author: "Sebastian"
authorLink: "https://github.com/buzzdeee"
description: "Eine Schritt-für-Schritt-Anleitung zum Überschreiben von Hugo-Layouts, um einen individuellen, mehrsprachigen Footer mit Support-Links zu erstellen."
license: "MIT"
images: [ "/images/tweaking-hugo-theme.png" ]

tags: [ "Hugo", "LoveIt", "Webentwicklung", "Tutorial", "Webdesign" ]
categories: [ "Anleitungen" ]

featuredImage: "/images/tweaking-hugo-theme.png"
featuredImagePreview: "/images/tweaking-hugo-theme.png"

# Eigene Parameter für unsere Logik
add_custom_footer: true

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
  enable: true
comment:
  enable: true
library:
  css:
  js:
seo:
  images: [ "/images/tweaking-hugo-theme.png" ]
---

## 1. Hugo Themes anpassen: Dein Blog, deine Regeln

Eine Website mit Hugo zu erstellen ist ein bisschen wie der Einzug in eine fertig möblierte Designer-Wohnung. Es sieht von dem Moment an großartig aus, in dem man die Tür öffnet, aber es fühlt sich erst dann wie ein richtiges *Zuhause* an, wenn man eigene Bilder an die Wände hängt oder den Teppich austauscht.

Das **LoveIt**-Theme ist wegen seiner cleanen Ästhetik, der integrierten Suche und dem Dark-Mode-Support extrem beliebt. Dennoch können generische Footer oder Standard-Layouts dazu führen, dass sich deine Ecke im Internet ein wenig nach "von der Stange" anfühlt. Egal, ob du einen individuellen Copyright-Hinweis, einen "Buy Me a Coffee"-Link oder spezifische Tracking-Skripte hinzufügen möchtest: Das Theme anzupassen ist der beste Weg, um deinem technischen Blog eine persönliche Note zu verleihen.

In dieser Anleitung gehen wir über die grundlegenden `hugo.toml`-Einstellungen hinaus und schauen uns an, wie du die zugrunde liegende Struktur sicher modifizierst, ohne die Möglichkeit zu verlieren, dein Theme später zu aktualisieren.

---

## 2. Ein Blick unter die Haube: Wie Hugo Themes (und Overrides) funktionieren

Bevor wir an den Code gehen, ist es wichtig, die **Hugo Lookup Order** (Suchreihenfolge) zu verstehen. Stell dir Hugo wie eine Suchmaschine vor, die nach Anweisungen sucht, wie deine Website gerendert werden soll.

### Die Hierarchie der Prioritäten
Wenn Hugo deine Website erstellt, sucht es nach Dateien (Layouts, Archetypes und Assets) in einer ganz bestimmten Prioritätsfolge:

1. **Dein Projekt-Root (`/layouts`, `/assets`):** Dies hat die höchste Priorität.
2. **Der Theme-Ordner (`/themes/LoveIt/layouts`, etc.):** Dies ist der Fallback (die Rückfallebene).

### Das "Override"-Prinzip
Die goldene Regel der Hugo-Entwicklung lautet: **Bearbeite niemals Dateien direkt im `themes/`-Ordner.** Wenn du die Theme-Dateien direkt änderst, werden deine Anpassungen gelöscht, sobald du das Theme über Git-Submodule oder einen Download aktualisierst. Stattdessen nutzen wir das **Override-Prinzip**:

* **Identifizieren:** Finde die Datei, die du ändern möchtest, innerhalb von `themes/LoveIt/layouts/...`.
* **Replizieren:** Erstelle die identische Ordnerstruktur in deinem **Root-Verzeichnis** (Hauptverzeichnis deines Projekts).
* **Modifizieren:** Kopiere die Datei in dein Root-Verzeichnis und bearbeite sie dort.

Da dein Root-Verzeichnis eine höhere Priorität hat, wird Hugo deine Version der Datei sehen und sagen: "Aha! Ich benutze diese angepasste Version anstelle der Standardversion aus dem Theme-Ordner."

## Was wir bauen
Bevor wir in den Code eintauchen, hier ein kurzer Überblick über das Custom-Footer-System, das wir für das **LoveIt**-Theme implementieren:

* **Mehrsprachige Unterstützung:** Zeigt Inhalte automatisch auf Englisch oder Deutsch an, basierend auf der Sprache des Beitrags.
* **Automatische Social-Media-Integration:** Verwendet die bereits in deiner `hugo.toml` definierten Links für PayPal, Patreon oder GitHub Sponsors wieder.
* **Theme-Konsistenz:** Nutzt die internen Plugins von LoveIt, damit Icons und Stile perfekt zu deiner Website passen.
* **Intelligente „Opt-Out“-Steuerung:** Der Footer erscheint standardmäßig bei allen Beiträgen, kann aber pro Beitrag deaktiviert werden, indem `add_custom_footer: false` in das Front Matter eingefügt wird.

### Schritt 1: Das Post-Layout überschreiben

Zuerst kopieren wir das Post-Template des Themes in unser Root-Verzeichnis. Bei LoveIt befindet sich dieses unter `themes/LoveIt/layouts/posts/single.html`. Kopiere es nach:
`layouts/posts/single.html`

In dieser Datei fügen wir einen Logik-Block an der Stelle ein, an der der Footer erscheinen soll (normalerweise nach dem Content-Bereich):

{{< highlight html "linenos=table" >}}
{{- /* Logik für den benutzerdefinierten Footer */ -}}
{{- if ne .Params.add_custom_footer false -}}
    <div class="post-footer-custom">
        <hr>
        {{- $footerPath := printf "custom/footer-%s.html" .Lang -}}
        {{- if templates.Exists (printf "_partials/%s" $footerPath) -}}
            {{- partial $footerPath . -}}
        {{- else -}}
            {{- /* Fallback auf Deutsch, falls die spezifische Sprachdatei fehlt */ -}}
            {{- partial "custom/footer-de.html" . -}}
        {{- end -}}
    </div>
{{- end -}}
{{< /highlight >}}

{{< admonition type="note" title="Opt-Out Steuerung" open=true >}}
Standardmäßig wird der Footer zu allen Beiträgen hinzugefügt. Die Prüfung `{{- if ne .Params.add_custom_footer false -}}` ermöglicht es dir, den Footer gezielt zu deaktivieren, wenn du `add_custom_footer: false` im Front Matter des jeweiligen Artikels setzt.
{{< /admonition >}}

{{< admonition type="tip" title="Technischer Hinweis: Umgang mit der „unsichtbaren“ Standardsprache" open=true >}}
Vielleicht ist dir aufgefallen, dass unsere Logik einen Fallback auf `footer-de.html` beinhaltet. Das ist Absicht!

In vielen Hugo-Setups ist die Standardsprache (in diesem Fall Deutsch) so konfiguriert, dass sie direkt im Root-Verzeichnis der URL erscheint (z. B. `example.com/titel-des-beitrags/`) und nicht unter einem sprachspezifischen Präfix wie `/de/`.

Da Hugo root-basierte Dateien nicht immer explizit mit einem Sprachpfad kennzeichnet (anders als bei übersetzten Unterverzeichnissen wie `/en/`), stellt dieser Fallback sicher, dass deine Leser auch dann deinen bevorzugten Standard-Footer sehen, wenn die Spracherkennung keinen eindeutigen String zurückgibt.
{{< /admonition >}}

### Schritt 2: Die Sprach-Partials erstellen

Erstelle nun den Inhalt für deinen Footer. Durch die Verwendung von `.Lang` erkennt Hugo automatisch, ob es sich um einen Beitrag unter `/en/` oder `/de/` handelt und wählt die entsprechende Datei aus.

Datei: `layouts/_partials/custom/footer-en.html`

{{< highlight html "linenos=table" >}}
<div class="custom-post-footer">
    <h4>Thoughts or Questions?</h4>
    <p>I'd love to hear your feedback! Feel free to share them below or leave a reaction.</p>
    <h4>Support My Work</h4>
    <div class="support-wrapper">
        {{- partial "custom/support-links.html" . -}}
    </div>
    <hr>
</div>
{{< /highlight >}}

Datei: `layouts/_partials/custom/footer-de.html`

{{< highlight html "linenos=table" >}}
<div class="custom-post-footer">
    <hr>
    <h4>Lust auf Austausch?</h4>
    <p>Hat dir dieser Beitrag geholfen? Ich freue mich riesig über dein Feedback, Fragen oder auch einfach ein kurzes „Hallo“ in den Kommentaren. Dein Input motiviert mich und hilft dabei, die Inhalte stetig zu verbessern!</p>

    <h4>Meine Arbeit unterstützen</h4>
    <p>Diesen Blog zu pflegen kostet Zeit und jede Menge Kaffee. Wenn dir meine Artikel gefallen, freue ich mich über eine kleine Unterstützung. Das hilft mir dabei, das Projekt werbefrei zu halten und regelmäßig neue Inhalte zu liefern!</p>

    <div class="support-wrapper">
        {{- partial "custom/support-links.html" . -}}
    </div>
</div>
{{< /highlight >}}

### Schritt 3: Die Social-Media-Konfiguration des Themes wiederverwenden

Das ist der „Pro-Move“: Anstatt Links fest im Code zu verdrahten, erstellen wir die Datei `layouts/_partials/custom/support-links.html`, um die Parameter direkt aus deiner `hugo.toml` zu ziehen.

Wir filtern deine bestehende Social-Media-Liste nach spezifischen „Support“-Keys (wie PayPal oder Patreon) und leiten diese an das interne Social-Plugin von LoveIt weiter. So bleibt das Styling konsistent mit dem Rest deines Themes:

{{< highlight html "linenos=table" >}}
<div class="support-links">
    {{- $socialMap := resources.Get "data/social.yml" | transform.Unmarshal -}}
    {{- $supportServices := slice "paypal" "patreon" "ko-fi" "githubsponsor" "liberapay" -}}

    {{- range $key, $value := .Site.Params.social -}}
        {{- if in $supportServices ($key | lower) -}}
            {{- $social := $key | lower | index $socialMap | default dict -}}
            {{- if $value -}}
                {{- if reflect.IsMap $value -}}
                    {{- $social = merge $social $value -}}
                {{- else -}}
                    {{- $social = dict "Id" $value | merge $social -}}
                {{- end -}}
                {{- partial "plugin/social.html" $social -}}
            {{- end -}}
        {{- end -}}
    {{- end -}}
</div>
{{< /highlight >}}

### Schritt 4: Styling – Alles kompakt halten

Damit die Überschriften und Absätze wie eine zusammengehörige Einheit wirken, füge Folgendes zu deiner `assets/css/_custom.scss` hinzu:

{{< highlight css "linenos=table" >}}
/* Styling für den benutzerdefinierten Post-Footer */
.custom-post-footer {
    margin-top: 2rem;
}

.custom-post-footer h4 {
    margin-bottom: 0.4rem; /* Abstand unter der Überschrift verringern */
    color: var(--title-color);
}

.custom-post-footer p {
    margin-top: 0; /* Oberen Abstand des Absatzes entfernen */
    margin-bottom: 1rem;
}

.support-wrapper {
    margin-top: 0.8rem;
    display: flex;
    gap: 12px;
}
{{< /highlight >}}

## Fazit

Indem wir das **Override-Prinzip** und Hugos `.Lang`-Variable genutzt haben, konnten wir einen dynamischen, mehrsprachigen Footer erstellen, der sich wie ein nativer Bestandteil des LoveIt-Themes anfühlt. Diese Lösung ist nicht nur sicher vor zukünftigen Theme-Updates, sondern bietet auch eine professionelle Möglichkeit, mit deinen Lesern in Kontakt zu treten und deine Arbeit zu unterstützen.

**Viel Spaß beim Codetweaking!**
