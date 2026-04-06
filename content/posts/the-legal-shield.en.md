---
title: "The Legal Shield: Setting Up Imprint and Privacy for a Multilingual Hugo Blog"
subtitle: "How I conquered the German 'Impressumspflicht' without exposing my home address."
date: 2026-04-03T22:00:48+02:00
lastmod: 2026-04-03T22:00:48+02:00
draft: false
author: ""
authorLink: ""
description: "A comprehensive guide on setting up a legally compliant, multilingual Hugo blog using ihr-impressum.de, Sipgate, and e-recht24. Includes technical Hugo tips for GDPR-compliant hosting."
license: "MIT"
images: [ "/images/the-legal-shield.png" ]

tags: [ "Hugo", "Privacy", "GDPR", "Blogging" ]
categories: [ "Web Development" ]

featuredImage: "/images/the-legal-shield.png"
featuredImagePreview: "/images/the-legal-shield.png"

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
  images: [ "/images/the-legal-shield.png" ]
  # ...
---

Starting a blog is exciting, but if you are hosting from Germany or targeting European readers, you'll hit a legal roadblock before your first "Hello World": the **Impressumspflicht** (Imprint) and the **DSGVO** (GDPR). 

Here is how I navigated the legal requirements, secured a privacy-friendly address, and implemented a multilingual setup in Hugo.

{{< admonition type=important title="Update: Newer Service Recommendation" open=true >}}
**Note:** Since writing this post, I have found a more efficient legal notice (Impressum) service. While the Hugo technical steps below are still valid, I recommend checking out my updated guide for the latest recommendation: **[Managed Imprint & MIT-Style Privacy: My Legal Setup for Hugo](/en/online-impressum/)**.
{{< /admonition >}}

## Why Do You Need This?

In Germany, § 5 DDG (formerly TMG) requires almost every website to have an "Impressum" (Legal Notice). It’s not just for businesses; even a hobby blog can be considered "business-like" if you have affiliate links or just a general "permanence."

**The catch:** You have to provide a physical address where you can be reached. For many (including me), putting a private home address on the public internet is a huge "no-go."

## Step 1: Getting a c/o Address

You have a few options to avoid using your home address:
1. **Friends/Family:** The "0€" route. You use their address with a `c/o` addition.
2. **Business Centers:** Professional but usually expensive (30€+ / month).
3. **Dedicated Imprint Services:** This is the sweet spot for bloggers.

I chose **[ihr-impressum.de](https://ihr-impressum.de)**. It’s affordable, provides a legally "ladungsfähige Anschrift" (a summons-ready address), and they handle the mail for you. But there are many others offering same/similar service to comparable prices.

{{< admonition type=info title="Ready to Use" open=true >}}
A major benefit of these services is that they usually provide a template or example legal notice. You can use these immediately with minimal tweaks, as the address and required legal disclosures are already perfectly aligned with their service.
{{< /admonition >}}

### Step 1.5: Phone and Email (The Digital Side)

Legal compliance doesn't stop at a physical address. According to § 5 DDG, you also need to provide a way for "fast electronic contact" and "direct communication." This means:

1. **A Phone Number:** To avoid giving out my private mobile number, I reactivated my old **Sipgate VoIP account**. It’s perfect because it's not tied to a physical landline, and people can leave a message on the voicemail which then gets forwarded to my email.
2. **A Dedicated Email:** I created a clean, professional address: `webmaster@reitenba.ch`. This keeps my personal inbox separate from any legal or administrative inquiries.

**Note:** If you use a VoIP service or a redirect, make sure the "direct communication" aspect is still somewhat given (e.g., check your voicemails regularly!).

**A note on Bot-Protection**: In my public imprint page, I display the address as `webmaster [at] reitenba.ch`. While humans understand this immediately, it makes it significantly harder for simple scraper bots to harvest the address for spam lists.

## Step 2: Generating the Privacy Policy

Privacy policies are a moving target. Instead of writing one yourself, use a generator. I used the free version of **[e-recht24.de](https://www.e-recht24.de)**. 

**Pro-tip:** Even if you think you don't track anything, remember that:
- **GitHub Pages** logs IP addresses.
- **GoatCounter** (if you use it) needs a mention.
- **Google Fonts/CDNs** often leak data to third parties (try to host them locally, or disable those).

## Step 3: Implementing Multilingual Support in Hugo

If your blog is bilingual (like mine), you need these pages in both languages. Here is the technical setup I used.

### The File Structure
Hugo uses a suffix system for languages. For my setup, I created:
- `content/impressum.md` (German)
- `content/imprint.en.md` (English)
- `content/datenschutz.md` (German)
- `content/privacy.en.md` (English)

{{< admonition type=note title="Konfiguration der Standardsprache" open=true >}}
In meiner Hugo-Instanz ist Deutsch als Standardsprache festgelegt. Daher benötigen die deutschen Dateien kein sprachabhängiges Suffix (wie `.de.md`), während die englischen Versionen zwingend das Suffix `.en.md` benötigen, um korrekt zugeordnet zu werden.
{{< /admonition >}}

### The Footer Logic
I wanted the footer to automatically link to the correct language version, therfore I built a "Partial Fallback" system.

In `layouts/partials/footer.html`, I added a call to a custom legal partial:

{{< highlight html "linenos=table" >}}
<div class="footer-line">
    {{- $legalPath := printf "custom/legal-%s.html" .Lang -}}
    {{- if templates.Exists (printf "_partials/%s" $legalPath) -}}
        {{- partial $legalPath . -}}
    {{- else -}}
        {{- /* Fallback to German */ -}}
        {{- partial "custom/legal-de.html" . -}}
    {{- end -}}
</div>
{{< /highlight >}}

Then, I created two small HTML snippets in `layouts/_partials/custom/`:

**legal-de.html**
{{< highlight html "linenos=table" >}}
<span><a href="/impressum/">Impressum</a></span>
&nbsp;|&nbsp;
<span><a href="/datenschutz/">Datenschutz</a></span>
{{< /highlight >}}

**legal-en.html**

{{< highlight html "linenos=table" >}}
<span><a href="/en/imprint/">Imprint</a></span>
&nbsp;|&nbsp;
<span><a href="/en/privacy/">Privacy Policy</a></span>
{{< /highlight >}}

## Step 4: Stripping Third-Party CDNs

To stay GDPR-compliant "by design," I disabled **jsDeliver** in the `hugo.toml`. This ensures that assets like Font Awesome are loaded directly from my GitHub Pages server, meaning no user IPs are leaked to other external networks.

{{< highlight toml "linenos=table" >}}
  # CDN config for third-party library files
  [params.cdn]
    data = "my_local.yml"
{{< /highlight >}}

And create an empty: `assets/data/cdn/my_local.yml` to override the default.

{{< admonition type=tip title="Connection Test" open=true >}}
Web developers should always use their browser's **Network Analysis Tools** (F12) after making such changes. Check exactly where the browser connects when loading your pages to ensure that no CDNs or other external sources are being loaded.
{{< /admonition >}}

## Conclusion

{{< admonition type=warning title="Disclaimer" open=true >}}
I am not a lawyer, and this post does not constitute legal advice. The legal requirements for websites (especially in Germany) are complex and subject to change. The information provided here reflects my personal experience and recommendations at the time of writing. For legally binding information, please consult a qualified legal professional.
{{< /admonition >}}

Legal compliance doesn't have to be expensive or ugly. With a service like `ihr-impressum.de`, a generator like `e-recht24.de`, and some Hugo logic, you can protect your privacy and your readers' data while looking professional.

Now, back to actual blogging!
