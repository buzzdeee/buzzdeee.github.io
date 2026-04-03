---
title: "Das Multi-Channel-Megaphon: Blog-Distribution automatisieren mit Hugo & Make.com"
subtitle: "Eine 5-Wege Social-Media-Pipeline via RSS und AT-Protokoll"
date: 2026-04-02T08:43:10+02:00
lastmod: 2026-04-02T08:43:10+02:00
draft: false
author: ""
authorLink: ""
description: "Ein Deep Dive in die Automatisierung deiner Blog-Beiträge auf Discord, LinkedIn, Facebook, Instagram und Bluesky mittels Hugo RSS-Overrides und Make.com."
license: "MIT"
images: [ "/images/the-multi-channel-megaphone.png" ]

tags: ["Hugo", "Automatisierung", "Make.com", "RSS", "API", "Bluesky", "Cloudinary", "Facebook", "Instagram", "Linkedin", "Discord", "DevOps", "Webentwicklung", "Tutorial"]
categories: [ "Tutorials"]

featuredImage: "/images/the-multi-channel-megaphone.png"
featuredImagePreview: "/images/the-multi-channel-megaphone.png"

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
seo:
  images: [ "/images/the-multi-channel-megaphone.png" ]
---

Ich liebe es, Code zu schreiben und OpenBSD-Ports zu pflegen, aber das manuelle Kopieren von Links auf fünf verschiedene Social-Media-Plattformen ist eine lästige Pflicht, die mich von meiner Werkbank fernhält. Hier zeige ich dir, wie ich ein „Zero-Touch“-Distributionssystem aufgebaut habe.

## Warum [Make.com](https://www.make.com/en/register?pc=buzzdeee)? (Der Automatisierungs-Vergleich 2026)

Als IT-Enthusiast möchte ich nicht nur, dass Dinge „funktionieren“ – ich möchte, dass sie effizient und kosteneffektiv sind. Bevor ich mich für dieses Setup entschieden habe, habe ich den aktuellen Automatisierungsmarkt sondiert. Warum habe ich [Make.com](https://www.make.com/en/register?pc=buzzdeee) gegenüber den anderen großen Playern den Vorzug gegeben?

### Der Marktvergleich 2026

| Plattform | Kostenlose Stufe (monatlich) | Der „Haken“ |
| :--- | :--- | :--- |
| **[Zapier](https://zapier.com/pricing)** | 100 Tasks | Teuer bei Skalierung; im kostenlosen Plan auf einfache 2-Schritt-„Zaps“ beschränkt. |
| **[IFTTT](https://ifttt.com/plans)** | 2 Applets | Super für IoT/Smart Home, aber zu starr für komplexe Logik. |
| **[Pipedream](https://pipedream.com/pricing)** | Credit-basiert | Gut für Entwickler, aber auf maximal 3 verknüpfte Accounts begrenzt. |
| **[n8n](https://github.com/n8n-io/n8n)** | Unbegrenzt (Self-hosted) | Der Goldstandard für Datenschutz, erfordert aber die Wartung eines eigenen Servers/Containers. |
| **[Make.com](https://www.make.com/en/pricing?pc=buzzdeee)** | **1.000 Operationen** | **Der Sweet Spot:** Visuelles Branching und genug Credits für einen vielbesuchten Blog. |

### Meine Gründe für [Make.com](https://www.make.com/en/register?pc=buzzdeee):

1. **Das „Operations“-Modell:** Im Gegensatz zu Zapier, das mehrstufige Workflows in der kostenlosen Stufe oft blockiert, erlaubt Make es dir von Tag eins an, massive, verzweigte „Scenarios“ mit Filtern und Routern zu bauen.
2. **Der 1.000-Credit-Puffer:** Indem ich meinen RSS-Trigger so einstelle, dass er nur einmal am Tag prüft, verbrauche ich nur **1 Operation pro Tag** für den Check. Selbst mit einer 5-Kanal-Verteilung (Discord, LinkedIn, FB, etc.) „kostet“ ein neuer Blogpost nur etwa 11–15 Operationen. In einem normalen Monat verbrauche ich damit kaum 20 % meines kostenlosen Kontingents.
3. **Visuelles Debugging:** Für jemanden, der den ganzen Tag im Terminal verbringt, ist ein visueller Flowchart, der genau zeigt, wo ein Post fehlgeschlagen ist (z. B. „Instagram hat dieses Bildformat abgelehnt“), ein echter Gamechanger. Es ist quasi ein GUI für meine Logik.

{{< admonition type="note" title="💡 Warum nicht \"n8n\"?" open=true >}}
Ich weiß, meine OpenBSD/Self-hosting-Kollegen schreien jetzt: „*Warum hostest du n8n nicht einfach auf deiner eigenen Hardware?*“

**Die Antwort**: Life-Balance. Zwischen dem Pflegen von Ports, dem Löten von Platinen und drei Kindern wollte ich einen Teil meiner Infrastruktur haben, den ich nicht patchen, absichern oder überwachen muss. [Make.com](https://www.make.com/en/register?pc=buzzdeee) ist mein „Managed Service“-Kompromiss – es funktioniert einfach, damit ich mich wieder dem echten Leben widmen kann.
{{< /admonition >}}

## Die Architektur: 1 Trigger, 5 Ziele

### Vorbereitung: Den Hugo RSS-Feed anpassen

Bevor [Make.com](https://www.make.com/en/register?pc=buzzdeee) seine Magie entfalten kann, müssen die Daten verfügbar sein. Standardmäßig liefern viele Hugo-Themes (wie *LoveIt*) einen sehr schlanken RSS-Feed aus, dem spezifische Metadaten für hochwertige Social-Media-Posts fehlen.

Um das zu beheben, habe ich einen **Override** für das RSS-Item-Template erstellt. Indem ich die `item.html` des Themes in meinen lokalen Ordner `layouts/_partials/rss/` kopiere, kann ich eigene Tags injizieren, die [Make.com](https://www.make.com/en/register?pc=buzzdeee) später auslesen kann.

#### Der Hugo Override (`layouts/_partials/rss/item.html`)

Ich habe benutzerdefinierte `<category>`, `<shortdesc>` und `<image>` Tags hinzugefügt, damit das „Featured Image“ und die spezifische Post-Beschreibung explizit im XML ausgegeben werden:

{{< highlight diff "linenos=table" >}}
--- themes/LoveIt/layouts/_partials/rss/item.html
+++ layouts/_partials/rss/item.html
@@ -37,4 +37,15 @@
           {{- $content | replaceRE `<figure[^>]*>.*</figure>` "" | replaceRE `<img[^>]*( /)?>` "" | safeHTML -}}
           {{- "]]>" | safeHTML -}}
       </description>
+    {{ range .Page.Params.tags -}}
+        <category>{{ . }}</category>
+    {{- end }}
+    <shortdesc>
+        {{ .Page.Params.description }}
+    </shortdesc>
+    <image>
+      {{ with .Page.Params.featuredImage }}
+        {{ . | absURL }}
+      {{ end }}
+    </image>
 </item>
{{< /highlight >}}

**Warum das Ganze?** Standard-RSS-Feeds „vergraben“ das Bild oft innerhalb des Beschreibungs-HTMLs. Durch den dedizierten `<image>`-Tag muss [Make.com](https://www.make.com/en/register?pc=buzzdeee) nicht raten oder die Seite crawlen – es erhält direkt die URL zum Beitragsbild, bereit für den Versand an Cloudinary oder Bluesky.

In jedem robusten System braucht man eine einzige **Source of Truth** (Quelle der Wahrheit). Für mich ist das die von Hugo generierte `index.xml`. Jedes Mal, wenn ich meine Seite deploye, erkennt die Make-Engine das Update und startet den Multi-Channel-Broadcast.

### 1. Der Trigger: RSS „Watch Items“
Das erste Modul ist der RSS-Watcher. Er prüft deinen Feed regelmäßig darauf, ob ein neuer `<item>`-Tag erschienen ist.
* **Der „Eco-Mode“-Tipp:** Im Jahr 2026 sind Automatisierungs-Credits wertvoll. Ich habe mein Intervall auf **einmal täglich (1440 Minuten)** eingestellt. Das ist der ultimative „Eco-Mode“ – es spart Operationen und verhindert versehentliches Spammen, falls du gerade nur schnelle CSS-Anpassungen an deiner Live-Seite vornimmst.

### 2. Discord (Der innere Zirkel)
**Modul:** *Discord > Post a Message*
Dies ist für meine treuesten Follower. Für diese Integration verwende ich keine veralteten Webhooks. Stattdessen nutze ich **OAuth**, um [Make.com](https://www.make.com/en/register?pc=buzzdeee) direkt mit meinem Discord-Konto zu verbinden, was wesentlich sicherer und einfacher zu verwalten ist.
* **Die Automatisierung:** Ich mappe einfach die RSS-`<URL>` in den Nachrichtentext. Die Engine von Discord übernimmt den Rest und sorgt automatisch für das „Unfurling“ der URL, um eine ansprechende Vorschau mit dem Titel und dem Vorschaubild anzuzeigen, das ich in meinen Hugo-Metadaten definiert habe.

### 3. LinkedIn (Das Karriere-Netzwerk)
**Modul:** *LinkedIn > Create a User Image Post*

LinkedIn ist die „professionelle Seite“ des Blogs, aber auch anspruchsvoll. Im Gegensatz zu Discord **generiert LinkedIn nicht zuverlässig** eine Rich-Preview aus einem einfachen Link. Wenn du einen Post willst, der die Leute wirklich zum Stoppen bringt, musst du die Daten manuell liefern.

* **Der Pro-Move:** Das ist genau der Grund, warum wir vorhin den Hugo-RSS-Feed angepasst haben. Ich verwende die spezifische `<image>`-URL aus dem Feed, um LinkedIn zu zwingen, das Beitragsbild anzuzeigen.
* **Der Inhalt:** Ich mappe den RSS-`<title>` und den `<shortdesc>`, den wir im XML bereitgestellt haben, direkt in den Textkörper des Posts. Das ergibt eine hochwertige, manuell wirkende Ankündigung, die die Schlagzeile, eine prägnante Zusammenfassung und das Visuelle enthält – alles automatisiert, aber mit dem Schliff eines handgemachten Posts.

### 4. Bluesky (Das offene Neuland)
**Update 2026:** Bluesky ist offiziell zum digitalen Dorfplatz für die Open-Source- und BSD-Community geworden. Hier finden die echten technischen Gespräche statt.

**Der Weg des „IT-Enthusiasten“:**
Obwohl [Make.com](https://www.make.com/en/register?pc=buzzdeee) ein komfortables „Create a Post“-Modul anbietet, gibt es einen frustrierenden Haken: Wählst du den Medientyp „Link“, werden die Metadaten nicht automatisch geladen; wählst du „Image“, erhältst du zwar ein schönes Bild, aber keinen anklickbaren Link zu deinem eigentlichen Blogpost.

Um das zu lösen, verwende ich das Modul **Bluesky > Make an API Call**. Es bietet die perfekte Balance: Es übernimmt den Authentifizierungs-Handshake für dich, überlässt den eigentlichen Aufbau des JSON-Payloads aber komplett dir. Ganz im Sinne von BSD lassen wir uns nicht von einer „Black Box“ einschränken, sondern kontrollieren die Datenstruktur selbst.

**Die Herausforderung: Kein automatisches Unfurling**
Bluesky generiert über die API keine automatischen Link-Vorschauen. Um eine Rich-Preview mit Bild zu erhalten, musst du das Embed-Objekt selbst bauen. Das erfordert eine Kette von vier Schritten in [Make.com](https://www.make.com/en/register?pc=buzzdeee):
1. **HTTP > Get a File:** Herunterladen des Beitragsbildes von der URL aus unserem angepassten RSS-Feed.
2. **Image > Resize:** Sicherstellen, dass das Bild klein genug für den Upload zu Bluesky ist.
3. **Bluesky > Upload Media:** Hochladen dieser Datei zu Bluesky, um einen „Blob“ (eine Referenz auf das Bild auf deren Servern) zu erhalten.
4. **HTTP > Make an API Call:** Senden des fertigen JSON-Payloads an die API.

**Details zum API-Aufruf:**
Ich nutze die `POST`-Methode an den Endpunkt `https://bsky.social/xrpc/com.atproto.repo.createRecord` mit folgendem JSON-Body:

{{< highlight json "linenos=table" >}}
{
  "repo": "buzzdeee.bsky.social",
  "collection": "app.bsky.feed.post",
  "record": {
    "text": "{{1.title}}",
    "createdAt": "{{now}}",
    "embed": {
      "$type": "app.bsky.embed.external",
      "external": {
        "uri": "{{1.url}}",
        "title": "{{1.title}}",
        "description": "{{1.rssFields.shortdesc}}",
        "thumb": {
          "$type": "blob",
          "ref": {
            "$link": "{{17.blob.ref.`$link`}}"
          },
          "mimeType": "{{17.blob.mimeType}}",
          "size": {{17.blob.size}}
        }
      }
    }
  }
}
{{< /highlight >}}

**Hinweis**: Die Referenzen wie `{{17.blob.size}}` und `{{17.blob.ref}}` sind dynamische Mappings in [Make.com](https://www.make.com/en/register?pc=buzzdeee), die auf die Ausgabe des „Upload Media“-Moduls (Modul 17 in meinem Szenario) verweisen. Dies stellt sicher, dass der Post perfekt mit dem hochgeladenen Thumbnail verknüpft ist.

#### Ein Wort zu X (ehemals Twitter):
Dir wird auffallen, dass X in dieser Liste fehlt. Um es direkt zu sagen: Das Posten auf X via API ist mittlerweile extrem mühsam geworden. Zwischen den ständig wechselnden API-Stufen und der Tatsache, dass grundlegende Automatisierung jetzt signifikante monatliche Gebühren allein für den Schreibzugriff kostet, habe ich mich entschieden, X zu ignorieren. Für einen Community-getriebenen Blog steht der Aufwand (ROI) in keinem Verhältnis zur offenen Natur von Bluesky.

### 5. Facebook Pages (Die professionelle Identität)
**Modul:** *Facebook Pages > Create a Post* – Beiträge werden automatisch mit Vorschau (Unfurling) erstellt.

Während mein Privatleben getrennt bleibt, lebt meine „professionelle Identität ;)“ als **buzzdeee** auf einer eigenen Facebook-Seite. Dies ist der sauberste Weg, um eine technische Marke von einem persönlichen Profil zu trennen.

**Der Kampf um den Namen:**
Wenn du jemals versucht hast, ein persönliches Facebook-Profil auf einen Handle wie „buzzdeee“ zu setzen, bist du wahrscheinlich der „Namenspolizei“ begegnet. Die Filter von Facebook sind berüchtigt dafür, sich wiederholende Zeichen und nicht-traditionelle Namen zu blockieren. **Pages** (Seiten) hingegen sind für Marken und Creator gedacht. Sie geben dir die Freiheit, deinen Handle genau so zu verwenden, wie du es möchtest – ohne den Zwang zu „Vorname / Nachname“.

### 6. Instagram (Der visuelle Part)
**Die Herausforderung: Die wählerischste API im Stack**

Instagram ist der visuellste Teil dieser Automatisierung, aber auch der restriktivste. Die API hat zwei harte Anforderungen: Sie akzeptiert nur **JPG-Dateien** (keine PNGs) und reagiert extrem empfindlich auf das Seitenverhältnis. Wenn dein Blog-Header ein breites 16:9 PNG ist, wird ein direkter Post einfach fehlschlagen.

**Die Lösung: Eine 5-stufige Verarbeitungskette**
Um eine 100-prozentige Erfolgsquote zu garantieren, habe ich in [Make.com](https://www.make.com/en/register?pc=buzzdeee) eine Pipeline gebaut, die das Asset vorbereitet, bevor Instagram es überhaupt zu Gesicht bekommt:

1. **HTTP > Download a File:** Lädt das Beitragsbild von der URL herunter, die unser benutzerdefinierter Hugo-`<image>`-Tag liefert.
2. **Image > Convert a Format:** Da mein Blog PNGs für optimale Schärfe nutzt, konvertiert dieses Modul die Datei in ein hochwertiges **JPG**, um die Instagram-API zufriedenzustellen.
3. **Cloudinary > Upload a Resource:** Lädt das konvertierte JPG in das Hochgeschwindigkeits-CDN von Cloudinary hoch.
4. **Cloudinary > Transform a Resource:** Das ist die „Geheimzutat“. Cloudinary nutzt KI-basiertes Cropping, um das Bild automatisch in ein perfektes **1080x1080 Quadrat** zu beschneiden und zu polstern. So bleiben Hardware-Details (wie eine Platine oder ein Skateboard) zentriert und das Seitenverhältnis ist konform.
5. **Instagram for Business > Create a Photo Post:** Schließlich greift dieses Modul auf die optimierte JPG-URL von Cloudinary zu und veröffentlicht den Post.

**Der Faktor „Cloudinary Free Tier“:**
Damit das funktioniert, benötigst du ein **kostenloses Cloudinary-Konto**. Ähnlich wie bei [Make.com](https://www.make.com/en/register?pc=buzzdeee) ist es großzügig, hat aber Limits, die man im Auge behalten sollte.
* **Credit-System:** Cloudinary nutzt „Credits“ für Speicher und Transformationen. 1 Credit entspricht in der Regel 1.000 Transformationen oder 1 GB verwaltetem Speicher.
* **Worauf zu achten ist:** Da wir für jeden einzelnen Blogpost ein Bild transformieren (beschneiden und konvertieren), verbrauchen wir pro Post nur einen winzigen Bruchteil eines Credits. Bei einem typischen Blog-Rhythmus wirst du das Limit wahrscheinlich nie erreichen, aber es lohnt sich, einmal im Monat das Dashboard zu prüfen, damit der Speicher nicht mit alten automatisierten „Temporär-Bildern“ vollgestopft wird.

**Der entscheidende Schritt: Der Meta-„Handshake“**
Bevor [Make.com](https://www.make.com/en/register?pc=buzzdeee) dein Konto sehen kann, musst du zwei spezifische administrative Aufgaben erledigen:
1. **Upgrade dein Instagram-Konto:** Gehe in deine Instagram-Einstellungen und wechsle den Kontotyp von **Persönlich** auf entweder **Creator** oder **Business**. Das ist kostenlos, aber der einzige Weg, um den API-Zugriff freizuschalten.
2. **Die Verknüpfung:** Verbinde dieses neue Konto in der Meta-Kontenübersicht oder über die Instagram-App mit deiner **Facebook-Seite**.

* **Das „Warum“:** Standardmäßige persönliche Konten haben keinen API-Zugriff für Automatisierungen. Durch das Upgrade und die Verknüpfung mit einer Seite „verbesserst“ du deinen Kontostatus in den Augen von Meta. Das erlaubt [Make.com](https://www.make.com/en/register?pc=buzzdeee), die Facebook-Seite als sicheres Gateway zu nutzen, um mit der Instagram Graph API zu kommunizieren.

> **Pro-Tipp:** Wenn du deinen Instagram-Handle in den Einstellungen deiner Facebook-Seite unter „Verknüpfte Konten“ siehst, hast du das API-Gateway für diese gesamte Kette erfolgreich freigeschaltet.

### Visualisierung der Pipeline

Zum Abschluss der technischen Analyse hilft ein Blick darauf, wie diese Module auf der Arbeitsfläche interagieren. Der folgende Screenshot bietet einen umfassenden Überblick über den gesamten Automatisierungs-Flow und illustriert den Datenpfad vom initialen RSS-Trigger bis hin zu den verschiedenen Transformations- und Distributionsstufen.

{{< figure
    src="/images/make_scenario.png"
    alt="Das komplette Szenario in make.com"
    caption="Das Szenario ist auf make.com [hier](https://eu1.make.com/public/shared-scenario/zVLytx8htAB/integration-rss-linked-in) geteilt"
    class="ma0 w-75"
>}}

# Fazit: Skalierbarkeit und zukünftige Automatisierungen

Der Aufbau dieser Pipeline ist erst der Anfang. Sobald du einen zuverlässigen Datenfluss von deiner Hugo-Seite zu deinen Social-Media-Kanälen hast, steht die Infrastruktur, um einen einfachen technischen Blog in ein vollautomatisiertes „Creator-Ökosystem“ zu verwandeln. Indem du deine Präsenz als vernetzten Stack betrachtest, kannst du damit beginnen, Interaktionen und sogar die Monetarisierung zu automatisieren.

#### Das Ökosystem erweitern
Wenn du skalieren möchtest, sind hier ein paar Ideen, in die du deine [Make.com](https://www.make.com/en/register?pc=buzzdeee)-Szenarien als Nächstes entwickeln kannst:

* **Automatisierte Interaktion (Der „Keyword“-Trigger):** Auf Plattformen wie Instagram kannst du ein „Watch Comments“-Modul einrichten. Wenn ein Nutzer einen Kommentar mit einem bestimmten Stichwort hinterlässt (z. B. „SKATE“ oder „CONFIG“), kann [Make.com](https://www.make.com/en/register?pc=buzzdeee) ihm automatisch eine Direktnachricht mit dem passenden Link zu deinem Blogpost oder einem Download schicken. Das steigert die Interaktionsrate und Conversions erheblich, ohne dass du auch nur einen Finger rühren musst.
    
* **Der „Sponsor“-Shoutout:**
    Wenn du Plattformen wie **Patreon**, **Ko-fi** oder **Buy Me a Coffee** nutzt, kannst du einen „Watch New Supporters“-Trigger hinzufügen. Wenn ein neues Trinkgeld oder ein Abo eingeht, kann [Make.com](https://www.make.com/en/register?pc=buzzdeee) sofort eine Dankeschön-Nachricht in einen speziellen `#sponsors`-Kanal in deinem **Discord** feuern oder sogar eine öffentliche „Willkommens“-Grafik auf deiner Facebook-Seite posten.

* **Automatisierte Content-Archivierung:**
    Jedes Mal, wenn ein Post erfolgreich abgesetzt wurde, könntest du die Pipeline anweisen, die Metadaten (URL, Datum, Performance-Stats) an ein **Google Sheet** oder eine **Airtable**-Basis anzuhängen. So erhältst du eine durchsuchbare Datenbank deiner Content-Historie für zukünftige „Throwback“-Posts oder Jahresrückblicke.

* **Plattformübergreifendes Recycling:**
    Du könntest sogar einen Filter setzen: Wenn ein Post auf LinkedIn besonders gut performt, wird nach einer Verzögerung von 48 Stunden automatisch ein „Re-Post“ auf einer anderen Plattform wie Reddit oder in einem technischen Forum ausgelöst.

#### Abschließende Gedanken
In der Welt von Open Source und BSD sprechen wir oft vom „richtigen Werkzeug für den Job“. [Make.com](https://www.make.com/en/register?pc=buzzdeee) wird – in Kombination mit einem gut formatierten RSS-Feed und ein wenig Cloudinary-Logik – genau zu diesem Werkzeug. Es ermöglicht dir, eine professionelle Multi-Channel-Präsenz zu pflegen, ohne die Zeit zu opfern, die du lieber mit Engineering oder Skaten verbringst.

Die anfängliche „Knochenarbeit“ mit den RSS-Overrides und API-Handshakes zahlt sich jedes Mal aus, wenn du `git push` ausführst. Sobald deine Pipeline steht, bleibt nur noch eines zu tun: Einfach weiter kreieren.

