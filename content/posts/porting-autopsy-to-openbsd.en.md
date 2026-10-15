---
title: "Frankensteining Autopsy on OpenBSD: A DFIR Porting Adventure"
subtitle: "From Dusty Java Myths, Outdated Bundles, and Fighting Apache Ivy's Network Appetite"
date: 2026-10-14T18:58:10+02:00
lastmod: 2026-10-14T18:58:10+02:00
draft: false
description: "A detailed account of porting Autopsy and its numerous dependencies such as Sleuthkit, OpenJFX, and SevenZipJBinding to OpenBSD."
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
mapbox:
share:
  enable: true
comment:
  enable: true
library:
  css:
  js:
seo:
  images: [ "/images/Autopsy_splash.png", "/images/Autopsy_main.png", "/images/Autopsy_timeline.png" ]
---

There is that old developer adage: *“Java: Build once, run everywhere.”* After my experiences over the last few months, I can only chuckle and say: "Keep dreaming" ;)

It all started late last year when I took over maintainership for a number of DFIR-related (Digital Forensics and Incident Response) security ports on OpenBSD. Early this year, the first major update was due—most notably [Sleuthkit](https://www.sleuthkit.org/) and [Plaso](https://github.com/log2timeline/plaso), which brought a considerable wave of new dependencies. Those command-line tools are great to have, but a proper GUI application was still missing: **[Autopsy](https://www.autopsy.com/)**.

The journey to port Autopsy began in the summer during a three-week rehab stay. The original plan of casually taking a Kaggle AI course turned out to be deadly boring. I needed a more hands-on, practical project. That’s when Autopsy came back to mind. Between rehab treatments, skateboarding, and hiking, it was a fitting pastime for the breaks in between ;) My Java knowledge was never stellar and, frankly speaking, rusty as hell—the perfect prerequisites for the motto: *What could possibly go wrong?*

## Anatomy of the Problem: Under the Hood of Autopsy

When you download and unpack the official Autopsy bundle, you quickly realize that its application architecture is heavily tailored toward Windows. The project bundles a massive amount of precompiled Windows binaries and DLLs—including tools like Plaso, 7-Zip, TestDisk, and GStreamer.

{{< admonition type="info" title="Precompiled Legacy Bundles" >}}
Many of the bundled binaries and JAR files (e.g., `sqlite-jdbc`, `openjfx`, `sevenzipjbinding`, or `sleuthkit`) are not only severely outdated in the upstream release, but also modified with custom patches. For non-Windows platforms, they usually only ship x86_64 Linux shared libraries—if anything at all.
{{< /admonition >}}

The core question was: Could I bend Autopsy to use native system programs built against OpenBSD shared libraries instead of relying on its bundled legacy components?

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

A wild porting spree ensued. Step by step, I ported, patched, and got the following dependencies running on OpenBSD:

* **`databases/sqlite-jdbc`**  
  *What it is:* The Java Database Connectivity (JDBC) interface for SQLite.  
  *What it does:* Autopsy uses SQLite to store and query case metadata, module results, and index databases. Since the JDBC driver relies on native C code via JNI, it had to be compiled natively against OpenBSD's system SQLite libraries.
* **`archivers/sevenzipjbinding`**  
  *What it is:* A Java binding for the 7-Zip C++ engine.  
  *What it does:* Allows Autopsy to directly inspect and extract compressed archives (ZIP, RAR, 7z) from evidence images. It turned out that Autopsy requires a specially patched version to handle compressed evidence smoothly.
* **`graphics/openjfx/21`**  
  *What it is:* The open-source framework for modern rich client UIs in Java.  
  *What it does:* Provides visual UI components. The embedded WebKit module and Media engine in OpenJFX are essential for rendering the interactive Timeline View in Autopsy as well as playing video and audio files directly within the GUI.
* **`sysutils/sleuthkit`** *(Java JNI Binding integrated/installed directly into the package)*  
  *What it is:* The underlying C/C++ forensics library and toolset for file system analysis (NTFS, FAT, ext4, etc.).  
  *What it does:* Autopsy uses Sleuthkit as its core engine for parsing disk images. Via the Java Native Interface (JNI), the Java frontend communicates directly with Sleuthkit's native C routines.
* **`sysutils/prefetch-parser`**  
  *What it is:* A Python tool and parsing framework for Windows Prefetch files.  
  *What it does:* Autopsy's *Recent Activity Ingest Module* originally relies on Mark McKinnon's *Artifact Parser* to decode Windows Prefetch artifacts and reveal program execution history. Autopsy bundles precompiled binaries for Windows, Linux, and macOS, but naturally nothing for OpenBSD. This is where `prefetch-parser` comes in as a performant OpenBSD alternative. I modified the ingest module accordingly to invoke this native system parser.

`openjfx` in particular turned out to be a tough nut to crack. To avoid poking around in the dark, I wrote small Java test programs to verify whether OpenJFX worked on OpenBSD in general or if the issue was within Autopsy's integration itself.

{{< figure
    src="/images/Autopsy_main.png"
    alt="Autopsy running on OpenBSD"
    caption="Autopsy running on OpenBSD."
    class="ma0 w-75"
>}}

At the end of the three weeks, I had a rather "frankensteined" Autopsy running on my OpenBSD box. But the results were impressive:

* **YARA Ingest Module:** Uses native system YARA (originally supported only on Windows).
* **OpenCV:** Uses OpenCV 4 from the OpenBSD Ports Tree (upstream Autopsy still uses OpenCV 2).
* **Plaso Ingest Module:** Uses the system-installed Plaso version (originally Windows-only).
* **Embedded Files Module:** Works seamlessly via the ported `sevenzipjbinding`.
* **Additional Integrations:** Native use of `testdisk`/`photorec`, `prefetch-parser` for recent activity analysis, working Timeline View thanks to OpenJFX WebKit, functioning media playback, and Solr keyword indexing.

## Autumn Polish: Feedback from the ports@ Mailing List

In September, it was time to clean up the new ports and prepare them for official import into the OpenBSD Ports Tree. The community feedback on the `ports@` mailing list was invaluable, though it required several iterations.

{{< admonition type="note" title="Post-Review Adjustments" >}}
* **sysutils/sleuthkit:** Originally planned as a subpackage (modeled after `opencv,-java`), but after discussion with `sthen@`, it was simplified and added directly to the `-main` package.
* **databases/sqlite-jdbc:** An earlier attempt existed. Together with fixes from Kurt Miller (`kurt@`), unversioned JARs were placed in `${MODJAVA_JAR_DIR}`. This convention was then applied to all other dependencies.
* **archivers/sevenzipjbinding:** Porting the official upstream version revealed that Autopsy strictly requires a custom-patched variant. Switching to it was fortunately not a big deal.
* **graphics/openjfx/21:** The biggest roadblock. Building the WebKit component required a massive set of patches. But it was worth the effort: the Timeline view works!
* **sysutils/prefetch-parser:** Thanks to `sysutils/libscca` already being in the ports tree, this import went smoothly.
{{< /admonition >}}

One key takeaway from `kurt@` (maintainer of the OpenBSD JDK ports) was that installing separate shared libraries wasn't necessary, as they are already bundled inside the respective `.jar` files. Additionally, all ports were moved out of `ports/java` into their appropriate subcategories.

Autopsy had to be adjusted accordingly, which meant minor Makefile tweaks and ultimately simplified the `autopsy.conf` configuration.

## Fighting Apache Ivy's Network Appetite

After tweaking the configurations, a real "Argh!" moment occurred during testing with `PORTS_PRIVSEP` (privilege separation during port builds): Autopsy kept trying to fetch dependencies from the internet—and quite a lot of them!

I spent an entire day trying in vain to list and prepare all dependencies in `DISTFILES` for offline compilation by Apache Ivy. After numerous struggles and dead ends, I decided to package the Ivy cache files downloaded during a build into a separate distfile:

{{< highlight sh >}}
# Temporarily open PF firewall for _pbuild
pfctl -f /etc/pf.conf.open

# Populate the Ivy cache in one run
make build

# Close the firewall again
pfctl -f /etc/pf.conf
{{< /highlight >}}

I zipped the cache, hosted it on GitHub, and instructed the port to extract it to the correct location in the `post-extract:` stage.

Naturally, success wasn't immediate (who would have guessed ;): One dependency used a *version range* instead of a pinned version, causing Ivy to check the network for updates despite having a populated local cache. After surgical edits to the cache resolving the range manually to an existing version, offline builds worked—at least briefly. Then I discovered the cache contains timestamps. Luckily, Ivy supports a parameter to bypass them:

{{< highlight properties >}}
-Divy.cache.ttl.default=eternal
{{< /highlight >}}

This forces Ivy to ignore cache expiration.

With that, the biggest roadblocks were cleared, and Autopsy was finally imported \o/.

## Chasing the Ghost Segfault

After importing Autopsy, one annoying cosmetic issue remained: Every time a new data source finished importing, Autopsy crashed with a segmentation fault and had to be restarted. Fortunately, the data source was saved successfully in the background and could be analyzed upon reopening. Not pretty, but better than being completely unusable.

With guidance from `kurt@`, who pointed me to the `debug-jdk` package, I began debugging. Something was corrupting the environment variables upon completing the import, causing subsequent `getenv()` calls to fail hard.

`kurt@` stayed on the case and shared helpful links overnight to relevant [Sleuthkit Pull Requests on GitHub](https://github.com/sleuthkit/sleuthkit/pull/3548). The root cause was identified: On POSIX systems, using `putenv()` instead of `setenv()` can corrupt the Java environment. 

`kurt@` then pointed out that the patch in the GitHub issue replaced `putenv()` with `setenv()` in additional places I had initially missed! So I took the complete patch from the issue—excluding the Mac-related XML chunks—and applied it to Sleuthkit.

However, the disappointment was immediate: Autopsy still crashed! *Doh!* The reason was simple: Autopsy bundled its own internal copy of Sleuthkit instead of linking dynamically to the updated system library. As a result, Autopsy itself needed a rebuild with a `REVISION` bump.

After that, everything worked smoothly (:

{{< figure
    src="/images/Autopsy_timeline.png"
    alt="Autopsy Timeline powered by OpenJFX WebKit on OpenBSD"
    caption="Autopsy Timeline powered by OpenJFX WebKit on OpenBSD."
    class="ma0 w-75"
>}}

## Summary and Outlook

When I started, I had no idea how far I would get or if anything usable would come out of it. LLMs proved to be helpful sparring partners for deciphering obscure error messages and brainstorming fixes.

The final result speaks for itself. In this configuration, OpenBSD actually offers better Autopsy support than Linux in certain areas—such as ingest modules that were previously Windows-exclusive. Additionally, the port benefits from much newer dependencies:

* **OpenCV 4** in OpenBSD ports vs. bundled OpenCV 2
* **Up-to-date Plaso** vs. ancient 2018 bundle
* **Latest Sleuthkit & SQLite-JDBC** versions

The effort was definitely worth it. Beyond gaining a working tool, the project provided deep insights into Autopsy's internals that I would have missed by using it purely on Windows.

Huge thanks to Kurt Miller (`kurt@`) for his deep Java expertise, reviews, patches, and testing across `i386`, `aarch64`, and `sparc64`. Without his help, this port wouldn't have been completed so quickly or cleanly.

**Key Takeaway?** 

Just because a project seems daunting or fails on the first, second, third, or fourth try is no reason to give up. It’s like skateboarding: try, try, try again, observe your mistakes, reflect, adjust, and when you fall—get back up and attempt it once more.

Since several ingest modules remain Windows-only, I'll likely take another look when time permits to see what else can be ported to OpenBSD.

