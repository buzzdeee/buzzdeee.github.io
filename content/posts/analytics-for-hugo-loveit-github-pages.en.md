---
title: "Analytics for Your Hugo Site: Choosing the Right Path"
subtitle: "Comparing Google Analytics, Plausible, Fathom, and GoatCounter for Hugo LoveIt"
date: 2026-03-28T09:24:51+01:00
lastmod: 2026-03-28T09:24:51+01:00
draft: false
author: ""
authorLink: ""
description: "A comprehensive guide to choosing and integrating analytics into a Hugo site using the LoveIt theme, featuring a deep dive into GoatCounter."
license: "MIT"
images: [ "/images/goat-watching-website-stats.png" ]

tags: ["Hugo", "LoveIt", "Analytics", "GoatCounter", "Privacy", "Google Analytics"]
categories: ["Tutorials", "Web Development"]

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
  images: [ "/images/goat-watching-website-stats.png" ]
  # ...
---

## Introduction

Running a blog or community page on GitHub Pages with Hugo is a masterclass in efficiency, but it does leave one gap: **visibility**. Because GitHub Pages is a static environment, we don't have access to server logs to see who is visiting our site. To understand our audience, we need to implement a client-side tracking solution.

In this post, we’ll compare the most popular analytics options for Hugo users and walk through the exact steps to enable them in the **LoveIt** theme.

## Comparing the Options

When choosing an analytics tool, you are essentially balancing three things: **Cost**, **Privacy**, and **Depth of Data**.
At the time of this writing, the LoveIt theme has built-in support for the following analytics tools:

1. **[Google Analytics (GA4)](https://analytics.google.com/)**
    * **The Vibe**: The industry standard. Massive, powerful, and slightly overwhelming.
    * **Pricing**: Free.
    * **Pros**: Incredible detail, free to use, and integrates with other Google tools.
    * **Cons**: Not privacy-focused (requires a cookie banner in many regions), heavy on page-load speed, and has a steep learning curve.

2. **[Plausible Analytics](https://plausible.io/)**
    * **The Vibe**: The "ethical" choice for developers. Clean, lightweight, and open-source.
    * **Pricing**: Paid (starting at approx. €9/mo) or free if you self-host on your own server.
    * **Pros**: No cookies required (GDPR/CCPA compliant), extremely fast, and the dashboard is a joy to use.
    * **Cons**: The hosted version costs money; lacks the deep "funnel" tracking of Google.

3. **[Fathom Analytics](https://usefathom.com/)**
    * **The Vibe**: Corporate-grade privacy. Extremely stable and professional.
    * **Pricing**: Paid (approx. $15/mo).
    * **Pros**: Highly compliant, ignores ad-blockers via custom domains, and very easy to set up.
    * **Cons**: Most expensive entry-level price for small hobby blogs.

4. **[GoatCounter](https://www.goatcounter.com)**
    * **The Vibe**: The "Goldilocks" solution. Simple, meaningful stats that don't treat your readers like products.
    * **Pricing**: Free for personal/non-commercial use; paid tiers for businesses.
    * **Pros**: Open-source, no cookies required (GDPR compliant), and can even work without JavaScript using a "tracking pixel."
    * **Cons**: The interface is very "text-heavy" and minimalist (no fancy graphs); requires a manual theme override to install.

5. **[Yandex Metrica](https://ads.yandex/metrica)**
    * **The Vibe**: The "free alternative" to Google with a superpower: Heatmaps.
    * **Pricing**: Free.
    * **Pros**: Includes "Session Replay" (watching a video of how users move their mouse) and Heatmaps for free—features that usually cost $100+/mo elsewhere.
    * **Cons**: Privacy concerns (data processed by Yandex); heaviest script on this list, which can slightly impact page load speed.

### Tabular comparison

| Feature | Google Analytics 4 | Plausible | Fathom | GoatCounter | Yandex Metrica |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Best For** | Power Users | Minimalists | Business/Privacy | Personal Blogs | Heatmaps/Session Replays |
| **Price** | **Free** | From €9/mo | From $15/mo | **Free** (Personal) | **Free** |
| **Privacy** | Low (Cookies) | High (Cookieless) | High (Cookieless) | High (Cookieless) | Low (Cookies) |
| **Setup** | Easy (Native) | Easy (Native) | Easy (Native) | Easy (Native) | Easy (Native) |
| **Script Weight**| ~30 KB | ~1 KB | ~2 KB | ~3.5 KB | ~45 KB |

{{< admonition type="tip" title="Note on Ad-Blockers & Accuracy" open=true >}}
Not all analytics are created equal when it comes to "invisibility." 
* **Google Analytics (GA4):** The most frequently blocked. Almost every ad-blocker and privacy-focused browser (like Brave or Firefox) stops Google's scripts by default. You may be "missing" 20-40% of your actual traffic.
* **Plausible & Fathom:** Better, but still occasionally blocked because their domains are well-known. However, they offer "Custom Domain" or "Proxy" setups that make their scripts look like part of your own site, bypassing most blockers.
* **GoatCounter:** Currently has a high "pass" rate because it's lightweight and doesn't track personal data, though it's still a JavaScript file that *can* be blocked.
* **The Solution:** To get 100% accurate data, you need to "hide" the tracker. You can do this by **Self-Hosting** the analytics on your own subdomain (e.g., `stats.yourdomain.com`) or using a **Server-Side** tool like Cloudflare.
{{< /admonition >}}

## Integrating analytics

Because the **LoveIt** theme is so well-built, you don't need to touch any code. Just choose your provider, get your ID/Code, and update your hugo.toml.

1. **Get Your Tracking Credentials**
    * **GA4**: Get your Measurement ID (G-XXXXXXXXXX).
    * **Plausible/Fathom**: Use your registered domain or ID.
    * **GoatCounter**: Use your "code" (the subdomain you picked, e.g., buzzdeee).
    * **Yandex**: Get your Counter ID (e.g., 12345678).

2. Configure `hugo.toml`

Open your configuration file and fill in the details for the service you want to enable:

{{< highlight toml "linenos=table,open=false" >}}
[params.analytics]
  enable = true
  # Google Analytics 4
  [params.analytics.google]
    id = "G-XXXXXXXXXX"
    # respect the browser’s “do not track” setting
    respectDoNotTrack = true
  # GoatCounter
  [params.analytics.goatCounter]
    code = "buzzdeee"
  # Plausible Analytics
  [params.analytics.plausible]
    dataDomain = "yourdomain.com"
  # Fathom Analytics
  [params.analytics.fathom]
    id = ""
    server = "" # Optional: for self-hosting
  # Yandex Metrica
  [params.analytics.yandexMetrica]
    id = ""
{{< /highlight >}}

## My Top Recommendation: GoatCounter 🐐

For a small to medium Hugo blog or community page, **GoatCounter** is the winner. It hits the perfect "Goldilocks" zone: it's as easy as Google to set up, as private as Plausible, but **free** for your personal site. It even lets you show off your page views to your readers, making your static site feel alive.

### Pro-Tip: Making the "Footer Counter" Appear

If you’ve added your GoatCounter code but don't see the "X views" text in your post footer, you likely missed a tiny but critical setting.

#### 1. Enable Analytics in `hugo.toml`

Even if you provide the GoatCounter code, the LoveIt theme won't execute the script unless the global analytics parameter is set to `true`. Make sure your config looks like this:

{{< highlight toml "linenos=table,open=false" >}}
[params.analytics]
  enable = true  # <--- THIS IS THE "MASTER SWITCH"
  [params.analytics.goatCounter]
    code = "your-subdomain" # e.g., "buzzdeee"
{{< /highlight >}}

#### 2. Flip the Switch in GoatCounter Settings

By default, GoatCounter protects your data and won't let your website "read" your stats to display them. You have to give it permission:

* Log in to your **GoatCounter Dashboard**.
* Go to **Settings** → **Data collection**.
* Check the box for: "**Allow adding visitor counts on your website**".
* Scroll down and hit **Save**.

#### Refresh and Verify

Once both are set, the theme's built-in script will fetch the data and inject it into the footer of your posts automatically. If it still doesn't show up, check your browser console (`F12`)—you might have an ad-blocker stopping the script while you are testing!

## Conclusion: Which One Should You Choose?

Selecting the right analytics tool for your Hugo site depends entirely on your goals for 2026:

* **The "Goldilocks" Choice (GoatCounter):** Perfect for personal bloggers. It’s free for reasonable use, respects privacy, and that built-in "Live View" counter in the LoveIt theme footer is a fantastic bonus.
* **The Professional Privacy Choice (Plausible or Fathom):** If you are running a business or a high-traffic site and want a "set it and forget it" solution with a beautiful, modern dashboard. Both offer a **1-month free trial**, so you can test them risk-free.
* **The Power User Choice (Google Analytics 4):** Best if you need deep integration with Google Ads or complex funnel tracking and don't mind the steeper learning curve (or the cookie banners).
* **The UX Designer Choice (Yandex Metrica):** If you specifically want to see **heatmaps** and watch recordings of how users interact with your site for free.

### Why I Recommend GoatCounter 🐐

For a Hugo blog, **GoatCounter** is my top pick. It hits the "Goldilocks" zone: it's as easy as Google to set up, as private as Plausible, and **free** for your personal site. It even lets you show off your page views to your readers, making your static site feel alive.

**Support the Developer:**
GoatCounter is an independent, open-source project. If you find it useful for your blog, consider supporting the creator via **[GitHub Sponsors](https://github.com/sponsors/arp242)**. It helps keep the service free for personal users and ensures the project stays independent!

## Final Checklist for Success

Before you hit `git push`, make sure you’ve covered these three critical bases:

1.  **The Master Switch:** Ensure `enable = true` is set under `[params.analytics]` in your `hugo.toml`. Without this, LoveIt won't load any of your tracking scripts!
2.  **GoatCounter API:** If you choose GoatCounter, remember to log into their dashboard and check the **"Allow adding visitor counts on your website"** box in the settings so your footer counter can actually fetch the data.
3.  **Respect Privacy:** If you go with a cookie-based tool like Google or Yandex, consider adding a simple Privacy Policy page to stay compliant with global regulations.

**Next Steps:** Pick your provider, grab your ID, and flip that `enable` switch to `true`. Your static blog isn't just a collection of files anymore—it's a living site with a measurable audience!

