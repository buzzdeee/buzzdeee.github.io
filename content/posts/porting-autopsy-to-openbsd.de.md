---
title: "Frankensteining Autopsy auf OpenBSD: Ein DFIR-Portierungsabenteuer"
subtitle: "Von verstaubten Java-Mythen, veralteten Bundles und dem Kampf gegen den Netzwerkhunger von Apache Ivy"
date: 2026-10-14T18:58:10+02:00
lastmod: 2026-10-14T18:58:10+02:00
draft: false
description: "Ein Erfahrungsbericht über die Portierung von Autopsy und seinen zahlreichen Abhängigkeiten wie Sleuthkit, OpenJFX und SevenZipJBinding auf OpenBSD."
license: "MIT"
images: [ "/images/Autopsy_splash.png", "/images/Autopsy_main.png", "/images/Autopsy_timeline.png" ]

tags:
  - openbsd
  - dfir
  - autopsy
  - java
  - ports
categories:
  - Security
  - OpenBSD

featuredImage: "/images/Autopsy_splash.png"
featuredImagePreview: "/images/Autopsy_splash.png"

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
  images: [ "/images/Autopsy_splash.png", "/images/Autopsy_main.png", "/images/Autopsy_timeline.png" ]
  # ...
---

Es gibt diesen alten Entwicklerspruch: *„Java: Build once, run everywhere.“* Nach meinen Erfahrungen in den letzten Monaten, kann ich den nur belächeln und sagen: "Träum weiter" ;)

Alles begann Ende letzten Jahres, als ich die Maintainerschaft für eine Reihe von DFIR-bezogenen (Digital Forensics and Incident Response) Security-Ports unter OpenBSD übernommen habe. Anfang des Jahres stand das erste große Update an – allen voran [Sleuthkit](https://www.sleuthkit.org/) und [Plaso](https://github.com/log2timeline/plaso), was eine beachtliche Welle neuer Abhängigkeiten mit sich brachte. Diese Kommandozeilentools sind gut zu haben, aber etwas mit GUI fehlt da noch: **[Autopsy](https://www.autopsy.com/)**

Der Beginn dann, Autopsy zu portieren, folgte im Sommer während eines dreiwöchigen Reha-Aufenthalts. Der ursprüngliche Plan, dort nebenbei entspannt einen KI-Kurs auf Kaggle zu absolvieren, erwies sich rasch als todlangweilig. Ich brauchte ein etwas handfesteres, praktischeres Projekt. Da kam mir Autopsy wieder in den Sinn. Zwischen Reha Behandlungen, Skateboarding, Wandern, ein angemessener Zeitvertreib für die Pausen dazwischen ;) Mein Java-Wissen war nie überragend und gelinde gesagt verdammt „rostig“ – also die perfekten Voraussetzungen für das Motto: *Was kann schon schiefgehen?*

## Die Anatomie des Problems: Ein Blick unter die Autopsy-Haube

Wer das offizielle Autopsy-Paket herunterlädt und entpackt, stellt schnell fest, dass die Anwendungsarchitektur stark auf Windows ausgerichtet ist. Das Projekt bündelt eine massive Menge an vorkompilierten Windows-Binaries und DLLs – darunter Tools wie Plaso, 7-Zip, TestDisk oder GStreamer.

{{< admonition type="info" title="Vorkompilierte Altlasten" >}}
Viele der gebündelten Binaries und JAR-Dateien (z. B. `sqlite-jdbc`, `openjfx`, `sevenzipjbinding` oder `sleuthkit`) sind im Upstream-Release nicht nur stark veraltet, sondern zum Teil auch mit speziellen Patches versehen. Für Nicht-Windows-Systeme bringen sie meist nur x86_64-Linux-Shared-Libraries mit – wenn überhaupt.
{{< /admonition >}}

Die Kernfrage lautete also: Kann ich Autopsy so hinbiegen, dass es statt der mitgelieferten Altlasten die nativen, gegen OpenBSD-Shared-Libraries gebauten System-Programme verwendet?

{{< highlight text >}}
+-------------------------------------------------------------+
|                     Autopsy GUI (NetBeans)                  |
+-------------------------------------------------------------+
                               |
       +-----------------------+-----------------------+
       |                       |                       |
       v                       v                       v
[ System Sleuthkit ]   [ System OpenJFX 21 ]  [ Native 7ZipJBinding ]
       |                       |                       |
       +-----------------------+-----------------------+
                               |
                               v
                     OpenBSD System Libraries
{{< /highlight >}}

Eine wilde Portierungs-Orgie nahm ihren Lauf. Schritt für Schritt habe ich folgende Abhängigkeiten portiert, gepatcht und auf OpenBSD zum Laufen gebracht:

* **`databases/sqlite-jdbc`**  
  *Was es ist:* Das Java Database Connectivity (JDBC) Interface für SQLite.  
  *Wozu es dient:* Autopsy nutzt SQLite zur Speicherung und Abfrage von Fallmetadaten, Modul-Ergebnissen und Indexdatenbanken. Da der JDBC-Treiber nativen C-Code über JNI nutzt, musste dieser nativ gegen die System-SQLite-Bibliotheken von OpenBSD kompiliert werden.
* **`archivers/sevenzipjbinding`**  
  *Was es ist:* Ein Java-Binding für die 7-Zip C++ Engine.  
  *Wozu es dient:* Ermöglicht Autopsy das direkte Durchsuchen und Extrahieren von komprimierten Archiven (ZIP, RAR, 7z) innerhalb von Datenträgern. Es stellte sich heraus, dass Autopsy eine speziell gepatchte Version benötigt, um reibungslos mit gepackten Beweismitteln umzugehen.
* **`graphics/openjfx/21`**  
  *Was es ist:* Das Open-Source-Framework für moderne Rich-Client-Benutzeroberflächen in Java.  
  *Wozu es dient:* Stellt visuelle Komponenten bereit. Insbesondere das eingebettete WebKit-Modul und die Media-Engine von OpenJFX sind essenziell, um die interaktive Timeline-View in Autopsy sowie Video- und Audio-Dateien direkt in der GUI zu rendern.
* **`sysutils/sleuthkit`** *(Java JNI Binding direkt im Paket integriert/installiert)*  
  *Was it ist:* Die zugrundeliegende C/C++ Forensik-Bibliothek und Werkzeugsammlung zur Dateisystem-Analyse (NTFS, FAT, ext4 etc.).  
  *Wozu es dient:* Autopsy nutzt Sleuthkit als Kern-Engine für das Parsen von Image-Dateien. Über das Java Native Interface (JNI) kommuniziert der Java-Frontend-Teil direkt mit den nativen C-Routinen von Sleuthkit.
* **`sysutils/prefetch-parser`**  
  *Was es ist:* Ein Tool und Parsing-Framework für Windows-Prefetch-Dateien.  
  *Wozu es dient:* Es dient im *Recent Activity Ingest Module* von Autopsy dazu, Windows-Prefetch-Spuren zu dekodieren und Aufschluss darüber zu geben, welche Programme wann auf dem analysierten Zielsystem ausgeführt wurden.
* **`sysutils/prefetch-parser`**  
  *Was es ist:* Ein Python Tool und Parsing-Framework für Windows-Prefetch-Dateien.
  *Wozu es dient:* Das *Recent Activity Ingest Module* von Autopsy verlässt sich im Original auf den *Artifact Parser* von Mark McKinnon, um Windows-Prefetch-Spuren zu dekodieren und Aufschluss darüber zu geben, welche Programme wann auf dem Zielsystem ausgeführt wurden. Mit Autopsy gebundelt sind Binaries für Windows, Linux und Mac, aber natürlich nichts für OpenBSD. Hier kommt jetzt der `prefetch-parser` als performante OpenBSD-Alternative zum Einsatz. Das Ingest-Modul wurde von mir entsprechend angepasst, um diesen nativen System-Parser aufzurufen.


Besonders `openjfx` erwies sich als zäher Brocken. Um nicht im Nebel zu stochern, habe ich kleine Java-Testprogramme geschrieben, mit denen ich isoliert prüfen konnte, ob OpenJFX auf OpenBSD prinzipiell funktioniert oder ob der Fehler in der Autopsy-Integration liegt.

{{< figure
    src="/images/Autopsy_main.png"
    alt="Autopsy unter OpenBSD"
    caption="Autopsy unter OpenBSD."
    class="ma0 w-75"
>}}

Am Ende der drei Wochen lief ein ziemlich „frankensteined“ Autopsy auf meiner OpenBSD-Maschine. Doch das Resultat konnte sich sehen lassen:

* **YARA Ingest Modul:** Nutzt das native System-YARA (im Original nur unter Windows verfügbar).
* **OpenCV:** Verwendet OpenCV 4 aus dem OpenBSD Ports Tree (Autopsy setzt standardmäßig noch auf OpenCV 2).
* **Plaso Ingest Modul:** Verwendet die Plaso-Version vom System (Original nur Windows).
* **Embedded Files Modul:** Funktioniert sauber über die portierte `sevenzipjbinding`.
* **Weitere Integrationen:** Native Nutzung von `testdisk`/`photorec`, `prefetch-parser` für Recent-Activity-Analysen, funktionsfähige Timeline View dank OpenJFX-WebKit-Part sowie funktionierende Medienwiedergabe und Solr-Keyword-Indexing.

## Feinschliff im Herbst: Feedback von der ports@ Mailingliste

Im September ging es an das Aufräumen der neuen Ports, um sie für den offiziellen Import in den OpenBSD Ports Tree vorzubereiten. Das Feedback der Community auf der `ports@` Mailingliste war extrem wertvoll, erforderte aber einige Überarbeitungen.

{{< admonition type="note" title="Anpassungen nach dem Review" >}}
* **sysutils/sleuthkit:** Ursprünglich als Subpackage geplant (orientiert an `opencv,-java`), nach Diskussion mit `sthen@` vereinfacht und direkt in das `-main` Paket integriert.
* **databases/sqlite-jdbc:** Hier gab es bereits einen älteren Versuch. Zusammen mit Korrekturen von Kurt Miller (`kurt@`) wurden die Unversioned-JARs in `${MODJAVA_JAR_DIR}` platziert. Dieses Schema wurde auf die anderen Abhängigkeiten übertragen.
* **archivers/sevenzipjbinding:** Nach der Portierung der offiziellen Upstream-Version zeigte sich, dass Autopsy zwingend eine custom-gepatchte Variante benötigt. Das Umstellen war zum Glück kein großer Act.
* **graphics/openjfx/21:** Der größte Roadblock. Insbesondere das Kompilieren des WebKit-Teils verlangte eine beträchtliche Menge an Patches. Belohnt wurde der Aufwand durch eine voll funktionsfähige Timeline-Ansicht.
* **sysutils/prefetch-parser:** Dank des bereits existierenden `sysutils/libscca` verlief dieser Import gewohnt reibungslos.
{{< /admonition >}}

Eines der wichtigsten Ergebnisse des Reviews durch `kurt@` (Maintainer der OpenBSD JDK-Ports) war die Feststellung, dass keine separaten Shared Libraries installiert werden müssen, da diese bereits gebündelt in den entsprechenden `.jar`-Dateien vorliegen. Zudem wanderten alle Ports aus `ports/java` in die jeweiligen spezifischen Subkategorien.

Daraufhin musste Autopsy erneut angepasst werden, was kleine Veränderungen im Makefile nach sich zog, am Ende aber zu einer deutlichen Vereinfachung der `autopsy.conf` führte.

## Der Kampf gegen den Netzwerkhunger von Apache Ivy

Nachdem die Konfigurationen angepasst waren, wartete beim Testen mit `PORTS_PRIVSEP` (Privilege Separation beim Port-Bau) noch ein echter Argh!-Moment: Autopsy versuchte beim Build weiterhin mnuter, Abhängigkeiten aus dem Internet nachzuladen – und das nicht zu knapp!

Einen ganzen Tag lang habe ich vergeblich versucht, sämtliche Abhängigkeiten in `DISTFILES` aufzulisten und vorzubereiten, damit Apache Ivy diese offline verwendet. Nach etlichen Verstrickungen und Misserfolgen auf diesem Wege, habe ich mich dann schlussendlich umentschieden, um die von Ivy heruntergeladenen Dateien im Ivy Cache als separates Distfile bereitzustellen, also:

{{< highlight sh >}}
# PF Firewall für _pbuild kurzzeitig öffnen
pfctl -f /etc/pf.conf.open

# Ivy-Cache in einem Durchgang befüllen
make build

# Firewall wieder schließen
pfctl -f /etc/pf.conf
{{< /highlight >}}

Dann den Cache zippen und auf GitHub hosten, und den Port so das er ihn im post-extract: Stage wieder an die richtige Stelle extrahiert.


Das war leider nicht mit sofortigem Erfolg gekrönt (Wer hätte das gedacht ;): Eine der Abhängigkeiten verwendete eine *Version Range* statt einer fest gepinnten Version, woraufhin Ivy trotz lokal gefülltem Cache wieder im Internet nach Updates suchen wollte. Erst nach chirurgischen Eingriffen im Cache und dem manuellen resolven der Range auf eine vorhandene Version gelang der Netzwerk-freie Build, für kurze Zeit zumindest. Bis ich dann feststellen musste, das der Cache mit Timestamps versehen ist. Zum Glück gibts den Ivy Parameter:

{{< highlight properties >}}
-Divy.cache.ttl.default=eternal
{{< /highlight >}}

Der Ivy diese Timestamps ignorieren lässt.

Damit waren die größten Roadblocks eliminiert, und Autopsy konnte endlich importiert werden \o/.

## Die Jagd nach dem Segfault

Nach dem erfolgreichen Import von Autopsy gab es da allerdings noch einen ziemlich nervigen kosmetischen Fehler: Jedes Mal, wenn das Hinzufügen einer neuen Datenquelle abgeschlossen war, verabschiedete sich Autopsy spontan mit einem Segmentation Fault und musste neu gestartet werden. Die Datenquelle war zum Glück im Hintergrund korrekt hinzugefügt worden und konnte nach dem Neustart problemlos analysiert werden. Nicht schön, aber immerhin besser als quasi unbenutzbar.

Mit der Unterstützung von `kurt@`, der mich auf das `debug-jdk`-Paket aufmerksam machte, ging es an die Ursachenforschung. Irgendetwas zerlegte da beim Import-Abschluss die Environment-Variablen, sodass ein anschließendes `getenv()` eiskalt fehlschlug.

`kurt@` blieb am Ball und schickte über Nacht hilfreiche Links zu entsprechenden [Sleuthkit Pull Requests auf GitHub](https://github.com/sleuthkit/sleuthkit/pull/3548). Die Wurzel des Übels war gefunden: Auf POSIX-Systemen kann die Verwendung von `putenv()` statt `setenv()` das Java-Environment korrumpieren. Ein minimalinvasiver Patch für Sleuthkit, der `putenv()` durch `setenv()` ersetzte, war dann nur noch eine leichte Fingerübung. `kurt@` wies mich im Verlauf darauf hin, dass der Patch am GitHub-Issue `putenv()` durch `setenv()` noch an weiteren Stellen ersetzte – ein Detail, das ich im ersten Anlauf schlicht übersehen hatte! Ich übernahm also den vollständigen Patch aus dem Issue, exclusive der Mac bezogenen Chunks zu den verschiedenen `.xml`-Dateien aus und wendete ihn auf Sleuthkit an.

Der Dämpfer folgte auf dem Fuße: Autopsy crashte nach dem Sleuthkit-Fix leider immer noch! *Doh!* Die Ursache war jedoch schnell durchschaut: Da Autopsy die alte Sleuthkit-Version intern gebündelt hat, anstatt dynamisch als Shared Object auf die bereits geupdatete System-Library zuzugreifen, verweilte das alte `putenv()` weiterhin im Paket. Also musste auch Autopsy mit einem `REVISION`-Bump neu gebaut werden. 

Danach war endlich alles schick (:

{{< figure
    src="/images/Autopsy_timeline.png"
    alt="Autopsy Timeline, powered by openjfx webkit, unter OpenBSD"
    caption="Autopsy Timeline, powered by openjfx webkit, unter OpenBSD."
    class="ma0 w-75"
>}}

## Fazit und Ausblick

Am Anfang hatte ich keine Ahnung, wie weit ich es hier bringen würde und ob überhaupt etwas Brauchbares am Ende herauskommen würde. Um aus Fehlermeldungen schlau zu werden und Ideen für Fixes zu sammeln, waren LLMs am Ende sehr hilfreich als Sparringspartner. 

Das Endergebnis kann sich mehr als sehen lassen. OpenBSD bietet für Autopsy in dieser Konfiguration an einigen Stellen sogar eine bessere Unterstützung als Linux – etwa bei Ingest-Modulen, die sonst nur unter Windows bereitstehen. Zudem profitiert das System von deutlich moderneren Abhängigkeiten:

* **OpenCV 4** im OpenBSD Ports Tree vs. gebündeltes OpenCV 2
* **Aktuelles Plaso** vs. veraltetes Bundle von 2018
* **Sleuthkit & SQLite-JDBC** jeweils auf dem neuesten Stand

Der Aufwand hat sich definitiv gelohnt. Neben einem funktionierenden Tool hat mir die Aktion tiefere Einblicke in die Architektur von Autopsy verschafft, die mir bei reiner Nutzung unter Windows verborgen geblieben wären.

Ein großer Dank geht an Kurt Miller (`kurt@`) für das profunde Java-Wissen, die detaillierten Reviews, Patches und Tests auf `i386`, `aarch64` und `sparc64`. Ohne diese Hilfe wäre der Port weder so schnell noch so sauber fertiggeworden.

**Was bleibt als Erkenntnis?** 

Bloß weil ein Vorhaben riesig und unmöglich erscheint oder beim ersten, zweiten, dritten oder vierten Mal nicht funktioniert, ist das kein Grund aufzugeben. Es ist wie beim Skaten: Probieren, probieren, probieren, sich selbst beobachten, reflektieren, weiter probieren – und wenn man stürzt: aufstehen und gleich nochmal versuchen.

Da diverse Ingest-Module immer noch nur für Windows verfügbar sind, werde ich bei Zeit und Gelegenheit schauen, ob man da nicht noch mehr nach OpenBSD portieren kann.

