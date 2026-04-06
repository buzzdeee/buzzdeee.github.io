---
title: "Managed Imprint & MIT-Style Privacy: My Legal Setup for Hugo"
subtitle: "Why I ditched static legal texts for a more flexible and privacy-focused approach."
date: 2026-04-04T10:00:00+02:00
lastmod: 2026-04-04T10:00:00+02:00
draft: false
author: ""
license: "MIT"
description: "A guide to my new legal setup for Hugo: Combining managed imprint hosting with a minimalist, cookie-free privacy policy – clean, secure, and maintenance-free."
images: ["/images/legal-shield-v2.png"]
tags: [ "Hugo", "Imprint", "Privacy", "GDPR", "Blogging" ]
categories: [ "Web Development" ]
featuredImage: "/images/legal-shield-v2.png"
seo:
  images: ["/images/legal-shield-v2.png"]
---

In my [previous article](/en/the-legal-shield/) on this topic, I described how to build a legally compliant imprint using a c/o address and VoIP numbers. While that works, it is technically "patchwork."

I have since found a solution that is even more elegant: **[online-impressum.de](https://online-impressum.de)**. Here is why I switched and how it simplifies my setup.

## The Problem with the Phone Number

According to § 5 DDG (German Digital Services Act), an imprint must provide means for "fast electronic contact" and "immediate communication." For a long time, this meant: Email + Phone. However, case law and the wording of the law allow for some leeway: if a **contact form** enables a response within a few minutes or hours, it can potentially replace the phone number.

## Why online-impressum.de?

Switching to this service—besides being more affordable—brought three decisive advantages that made my old setup (Sipgate + Mail Service) redundant:

### 1. External Hosting (Managed Imprint)

Instead of maintaining legal information statically in Hugo partials or rebuilding the entire project every time your address changes, you can use an external host like `www.mein.online-impressum.de/<your-pseudonym>`.

* **Centralized Management:** Legal changes or updates to your address are automatically handled by the provider. You simply link to this dynamic URL from your Hugo blog.
* **Cross-Platform Solution:** This is particularly practical if you are active on social media. Since you often cannot host your own legal subpage there, the external link serves as a central, legally secure landing page for all your profiles.

### 2. The Pseudonym Email
You receive an email address following the pattern `<pseudonym>@mail.online-impressum.de`.
* This strictly separates private and business inquiries.
* Emails are reliably forwarded without your real address appearing in the page's source code (protection against scraping).

### 3. The Integrated Contact Form
This is the real "highlight." Since online-impressum.de offers an integrated contact form on your profile page, sufficient communication channels are provided.
**The result:** I no longer need to list a (VoIP) phone number in my imprint. This increases the barrier for phone spam and massively protects my privacy.

### Optional: The Privacy Package
For those looking for an all-in-one solution: online-impressum.de also offers a **Privacy Package add-on**. This allows your Privacy Policy to be hosted centrally and kept up-to-date alongside your imprint.

I currently only use the Imprint Package because my Hugo privacy setup is already established—but if you are just starting out or want to save yourself the maintenance of GDPR texts, this is an elegant "all-in-one" option.

## The "Golden Way": Maximum Security Despite the Gray Area

A critical point with many imprint services is the phone number. The provider claims that the contact form suffices as a second communication channel. While this can work, it remains a slight legal gray area since German courts are often very strict regarding "immediate availability" (§ 5 DDG).

If you—like me—want to avoid any risk, choose the safest combination:

### Pro-Tip: Sipgate + Online-Impressum
At **online-impressum.de**, you have full control over your data. In the settings, you can:
* **Add your VoIP number:** I simply entered my Sipgate number there. It then appears perfectly in the imprint.
* **Control the form:** You can use the contact form as an additional channel or deactivate it entirely if you only want to offer phone and email.

**My current setup:** I use the provider's hosting for the legally required address but have added my Sipgate number with a voicemail-to-email function. This makes the imprint absolutely "watertight" without my private cell phone ever ringing.

{{< admonition type=tip title="Safety First" open=true >}}
The combination of a VoIP number (for legal immediacy) and the managed service (for the summons-ready address) is currently the cleanest solution to balance anonymity and legal requirements.
{{< /admonition >}}

## Integration in Hugo

Technical integration is now even easier than before. Instead of local Markdown files for the imprint, I simply adjust my footer logic.

In `layouts/_partials/custom/legal-de.html` (or other language files):

{{< highlight html "linenos=table" >}}
<span><a href="https://www.mein.online-impressum.de/buzzdeee" rel="noopener" target="_blank">Imprint</a></span>
{{< /highlight >}}

## Imprint Obligations for Social Media Accounts

If you use social networks for more than purely private purposes—such as building reach for a business, blog, or brand—you are subject to the **Imprint Obligation** in Germany (according to § 5 DDG). As soon as an account is conducted "commercially"—which applies to most influencers, freelancers, or companies—provider information must be easily recognizable, directly accessible, and constantly available.

The legal problem: Many platforms like Instagram or TikTok do not provide a dedicated field for legal texts. This is where the so-called **"Two-Click Rule"** applies: A user must be able to reach your full imprint from your profile with a maximum of two clicks. Furthermore, the link must be clearly labeled (e.g., "Imprint").

To avoid the risk of a warning (Abmahnung), the link should be integrated directly into your bio. You can find a detailed overview with step-by-step instructions for various channels here:

> 🔗 **[Social Media Setup: Instructions for Key Channels](https://online-impressum.de/blog/socialmedia-setup/)**

## Privacy: Quality over Quantity

In my original setup, I used a free privacy policy generator from e-recht24. It’s great for getting started, but it has one major disadvantage: these generators often spit out a giant "text monster" filled with clauses for things that don't even exist on a minimalist Hugo blog.

If you—like me—value a "short and sweet" **MIT License** over a heavy GPL, you probably want the same for your legal texts.

### My Minimalist GDPR Approach
Since my Hugo blog uses no cookies, avoids social media plugins, and only embeds external resources (like YouTube) in `privacyEnhanced` mode, I have massively streamlined the policy.

**Why this is better:**
* **Clarity:** Visitors immediately find the information they are looking for (e.g., regarding the privacy-friendly **GoatCounter**).
* **Honesty:** A text explaining 50 services you don't use feels more like an "alibi text" than actual information.
* **Maintainability:** Less text means less work during legal updates.

Instead of wading through 20 pages of "maybe-clauses," I generated a precise policy tailored to my lean stack. This fits much better with the spirit of a static site generator: **Fast, efficient, and only the essentials.**

{{< admonition type=info title="Checklist for Hugo Purists" open=true >}}
If you don't use tracking (except perhaps GoatCounter/Plausible) and no CDNs, your privacy policy should reflect that. A short, precise text is often legally more valuable than a generic 30-page document that doesn't fit the website.
{{< /admonition >}}

## Conclusion: Why the Switch Makes Sense for Me

Anyone professionalizing their blog or social media presence shouldn't get bogged down with technical "patchwork" regarding legal texts. Switching to a managed service like **online-impressum.de** has simplified my setup while making it more secure.

* **Privacy Protected:** No private address or (VoIP) phone number exposed to the public thanks to the mail service and the option to cleanly integrate a Sipgate number.
* **SaaS Advantage (Managed Hosting):** Once linked in the Hugo footer, the blog never needs to be rebuilt for legal updates—changes are managed centrally by the provider.
* **Social-Media-Ready:** Thanks to the dedicated landing page, you have a clean link for your bio (Instagram, TikTok, etc.) that satisfies the "Two-Click Rule."
* **Maintenance-Free:** Legal changes (like the transition from TMG to DDG) are implemented automatically in the background.

### "MIT-Style" for Privacy too
Fitting this lean setup, I've also put my [privacy policy](/en/privacy/) on a diet. Instead of 20 pages of standard clauses, my policy is now exactly how I like my software licenses: **Short, precise, and to the point.** If you don't set cookies and don't use third-party CDNs, you don't need a text monster. My privacy policy now reflects exactly what my Hugo stack technically does—nothing more, nothing less. This is "Privacy by Design" at its best.

{{< admonition type="info" title="Transparency Note" >}}
I am not affiliated with online-impressum.de; I am simply a satisfied customer. If you want to order your imprint there as well, you can use my affiliate code **7364** during checkout. This supports my work on this blog a little—at no extra cost to you, of course.
{{< /admonition >}}
