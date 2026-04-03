---
title: "The Multi-Channel Megaphone: Automating My Hugo Blog Distribution with Make.com"
subtitle: "Engineering a 5-Way Social Media Pipeline via RSS and the AT Protocol"
date: 2026-04-02T08:43:10+02:00
lastmod: 2026-04-02T08:43:10+02:00
draft: false
author: ""
authorLink: ""
description: "A deep dive into automating blog post distribution across Discord, LinkedIn, Facebook, Instagram, and Bluesky using Hugo RSS overrides and Make.com."
license: "MIT"
images: [ "/images/the-multi-channel-megaphone.png" ]

tags: ["Hugo", "Automation", "Make.com", "RSS", "API", "Bluesky", "Cloudinary", "Facebook", "Instagram", "Linkedin", "Discord", DevOps", "Web Development", "Tutorial"]
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
  images: [ "/images/the-multi-channel-megaphone.png" ]
  # ...
---

I love writing code and maintaining OpenBSD ports, but manually copying links to five different social media platforms is a chore that keeps me away from my workbench. Here’s how I built a 'zero-touch' distribution system.

## Why [Make.com](https://www.make.com/en/register?pc=buzzdeee)? (The 2026 Automation Shootout)

As an IT enthusiast, I don’t just want things to "work" - I want them to be efficient and cost-effective. Before committing to this setup, I audited the current automation market. Why did I choose Make over the other major players?

### The 2026 Market Comparison

| Platform | Free Tier (Monthly) | The "Catch" |
| :--- | :--- | :--- |
| **[Zapier](https://zapier.com/pricing)** | 100 Tasks | Expensive at scale; limited to simple 2-step "Zaps" on the free plan. |
| **[IFTTT](https://ifttt.com/plans)** | 2 Applets | Great for IoT/Smart Home, but too rigid for complex logic. |
| **[Pipedream](https://pipedream.com/pricing)** | Credit-based | Seems to be good for developers, but maximum of 3 accounts to be used. |
| **[n8n](https://github.com/n8n-io/n8n)** | Unlimited (Self-hosted) | The Privacy Gold Standard, but requires maintaining your own server/container. |
| **[Make.com](https://www.make.com/en/pricing?pc=buzzdeee)** | **1,000 Operations** | **The Sweet Spot:** Visual branching and enough credits for a high-volume blog. |

### My Logic for Choosing [Make.com](https://www.make.com/en/register?pc=buzzdeee):

1.  **The "Operations" Model:** Unlike Zapier, which often blocks multi-step workflows on its free tier, Make allows you to build massive, branching "Scenarios" with filters and routers from day one.
2.  **The 1,000 Credit Buffer:** By setting my RSS trigger to poll once a day, I only use **1 operation per day** for the check. Even with a 5-channel distribution chain (Discord, LinkedIn, FB, etc.), a new blog post only "costs" about 11–15 operations. In a standard month, I’m barely touching 20% of my free allowance.
3.  **Visual Debugging:** For someone who spends all day in the terminal, seeing a visual flowchart that shows exactly where a post failed (e.g., "Instagram rejected this image ratio") is a game-changer. It is a GUI for my logic.

{{< admonition type="note" title="💡 Why not \"n8n\"?" open=true >}}
I know my fellow OpenBSD/Self-hosting fans are screaming, "*Why not just host n8n on your own metal?*"

**The Answer**: Life balance. Between maintaining ports, soldering PCBs, and three kids, I wanted one part of my infrastructure that I didn't have to patch, secure, or monitor. [Make.com](https://www.make.com/en/register?pc=buzzdeee) is my "Managed Service" compromise - it just works so I can get back to real life.
{{< /admonition >}}

## The Architecture: 1 Trigger, 5 Destinations

### Preparation: Tweaking the Hugo RSS Feed

Before [Make.com](https://www.make.com/en/register?pc=buzzdeee) can work its magic, the data needs to be available. By default, many Hugo themes (like *LoveIt*) provide a lean RSS feed that lacks the specific metadata needed for high-quality social media posts. 

To fix this, I created an **override** for the RSS item template. By copying the theme's `item.html` into my local `layouts/_partials/rss/` folder, I can inject custom tags that [Make.com](https://www.make.com/en/register?pc=buzzdeee) will later "pick up."

#### The Hugo Override (`layouts/_partials/rss/item.html`)

I added custom `<category>`, `<shortdesc>`, and `<image>` tags so that the "featured image" and the specific post description are explicitly exposed in the XML:

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

**Why do this?** Standard RSS feeds often bury the image inside the description HTML. By creating a dedicated <image> tag, [Make.com](https://www.make.com/en/register?pc=buzzdeee) doesn't have to "guess" or scrape the site—it gets the direct URL to the featured image, ready to be sent to Cloudinary or Bluesky.

In any robust system, you need a single **Source of Truth**. For me, it’s the `index.xml` file generated by Hugo. Every time I deploy my site, the Make engine detects the update and kicks off the multi-channel broadcast.

### 1. The Trigger: RSS "Watch Items"
The first module is the RSS Watcher. It polls your feed to see if a new `<item>` tag has appeared.
* **The "Eco-Mode" Tip:** In 2026, automation credits are precious. I set my interval to **once a day (1440 minutes)**. It’s the ultimate "Eco-mode"—it saves operations and prevents accidental spamming if you’re making quick CSS tweaks to your live site.

### 2. Discord (The Inner Circle)
**Module:** *Discord > Post a Message*
This is for my most loyal followers. This is for my most loyal followers. For this integration, I don't use old-school webhooks. Instead, I use **OAuth** to connect [Make.com](https://www.make.com/en/register?pc=buzzdeee) directly to my Discord account, which is much more secure and easier to manage.
    * **The Automation**: I simply map the RSS `<URL>` to the message content. Discord’s engine takes over from there, automatically "unfurling" the URL to show a rich preview with the title and the cover image I've defined in my Hugo metadata.

### 3. LinkedIn (The Career Network)
**Module:** *LinkedIn > Create a User Image Post*

LinkedIn is the "Professional Side" of the blog, but it’s also demanding. Unlike Discord, LinkedIn **does not reliably unfurl** a plain link to show a rich preview. If you want a post that actually stops the scroll, you have to provide the data manually.

* **The Pro Move:** This is exactly why we tweaked the Hugo RSS feed earlier. I use the specific `<image>` URL from the feed to force LinkedIn to show the featured image.
* **The Content:** I map the RSS `<title>` and the `<shortdesc>` we exposed in the XML directly into the post body. This creates a high-quality, manual-looking announcement that includes the headline, a concise summary, and the visual—all automated, but with the polish of a hand-crafted post.

### 4. Bluesky (The Open Frontier)
**The 2026 Update:** Bluesky has officially become the digital town square for the Open Source and BSD community. It’s where the real technical conversations are happening.

**The "IT Enthusiast" Way:**
While [Make.com](https://www.make.com/en/register?pc=buzzdeee) offers a high-level "Create a Post" module, it presents a frustrating trade-off: if you choose the "Link" media type, it doesn’t automatically unfurl the metadata; if you choose the "Image" type, you get a nice picture but no clickable link to your actual blog post. 

To solve this, I use the **Bluesky > Make an API Call** module. It strikes the perfect balance: it handles the authentication handshake for you, but leaves the actual construction of the JSON payload entirely in your hands. In true BSD fashion, we aren't letting a "black box" limit our output; we are controlling the data structure ourselves.

**The Challenge: No Automatic Unfurling**
Bluesky does not "unfurl" links automatically via the API. To get a rich preview with an image, you have to build the embed object yourself. This requires a three-step chain in [Make.com](https://www.make.com/en/register?pc=buzzdeee):
1.  **HTTP > Get a File:** Download the featured image from the URL provided by our tweaked RSS feed.
2.  **Image -> Resize:** Ensure the image is small enough, to be uploaded to Bluesky.
3.  **Bluesky > Upload Media:** Upload that file to Bluesky to receive a "blob" (a reference to the image on their servers).
4.  **HTTP > Make an API Call:** Send the final JSON payload to the API.

**The API Call Details:**
I use the `POST` method to the endpoint `https://bsky.social/xrpc/com.atproto.repo.createRecord` with the following JSON body:

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

**Note**: The references like `{{17.blob.size}}` and `{{17.blob.ref}}` are dynamic mappings in [Make.com](https://www.make.com/en/register?pc=buzzdeee) that point to the output of the "Upload Media" module (Module 17 in my scenario). This ensures the post is perfectly linked to the uploaded thumbnail.

#### A Word on X (formerly Twitter):
You might notice X is missing from this list. To put it bluntly: posting to X via API has become a massive PITA. Between the constantly shifting API tiers and the fact that basic automation now costs a significant amount of monthly $$$ just for "Write" access, I’ve decided to skip it. For a community-driven blog, the ROI simply isn't there compared to the open nature of Bluesky.

### 5. Facebook Pages (The Professional Identity)
**Module:** *Facebook Pages > Create a Post* Posts are automatically unfurled.

While my personal life stays separate, my "Professional Identity ;)" as **buzzdeee** lives on a dedicated Facebook Page. This is the cleanest way to separate a technical brand from a personal profile.

**The Name Battle:**
If you’ve ever tried to set a personal Facebook profile name to a handle like "buzzdeee," you’ve likely met the "Name Police." Facebook's filters are notorious for blocking repeating characters and non-traditional names. **Pages**, however, are designed for brands and creators. They give you the freedom to use your handle exactly as intended, with no "First Name / Last Name" requirement.

#### 6. Instagram (The Visual Maker)
**The Challenge: The Pickiest API in the Stack**

Instagram is the most visual part of this automation, but it’s also the most restrictive. The API has two hard requirements: it only accepts **JPG** files (no PNGs), and it is extremely sensitive to aspect ratios. If your blog header is a wide 16:9 PNG, a direct post will simply fail.

**The Solution: A 5-Step Processing Chain**
To ensure a 100% success rate, I built a specific pipeline in [Make.com](https://www.make.com/en/register?pc=buzzdeee) that preps the asset before Instagram even sees it:

1.  **HTTP > Download a File:** Grab the featured image from the URL provided by our custom Hugo `<image>` tag.
2.  **Image > Convert a Format:** Since my blog uses PNGs for clarity, this module converts the file into a high-quality **JPG** to satisfy the Instagram API.
3.  **Cloudinary > Upload a Resource:** Upload the converted JPG to Cloudinary’s high-speed CDN.
4.  **Cloudinary > Transform a Resource:** This is the "Secret Sauce." Cloudinary uses AI-based gravity to automatically crop and pad the image into a perfect **1080x1080 square**, ensuring the hardware details (like a PCB or a skateboard) stay centered and the aspect ratio is compliant.
5.  **Instagram for Business > Create a Photo Post:** Finally, this module references the optimized JPG URL from Cloudinary and publishes the post.

**The Cloudinary "Free Tier" Factor:**
To make this work, you’ll need a **Free Tier Cloudinary account**. Much like [Make.com](https://www.make.com/en/register?pc=buzzdeee), it’s generous but has limits you need to monitor.
* **Credit System:** Cloudinary uses "Credits" for storage and transformations. 1 Credit usually equals 1,000 transformations or 1GB of managed storage.
* **What to Watch Out For:** Since we are transforming an image for every single blog post (cropping and converting), we use a small fraction of a credit per post. For a typical blog posting schedule, you will likely never hit the ceiling, but it’s worth checking your dashboard once a month to ensure your storage isn't filling up with old automated "temp" images.

**The Crucial Step: The Meta "Handshake"**
Before [Make.com](https://www.make.com/en/register?pc=buzzdeee) can see your account, you need to perform two specific administrative tasks:
1.  **Upgrade your Instagram Account:** Go into your Instagram settings and switch your account type from **Personal** to either **Creator** or **Business**. This is free, but it's the only way to unlock API access.
2.  **The Link:** Connect that newly upgraded account to your **Facebook Page** in the Meta Accounts Center or via Instagram mobile app.

* **The "Why":** Standard personal accounts don't have API access for automation. By upgrading and linking to a Page, you "upgrade" your account status in Meta's eyes, allowing [Make.com](https://www.make.com/en/register?pc=buzzdeee) to use the Facebook Page as a secure gateway to talk to the Instagram Graph API.

> **Pro-Tip:** If you see your Instagram handle listed under "Linked Accounts" in your Facebook Page settings, you’ve successfully unlocked the API gateway for this entire chain.

### Visualizing the Pipeline

To conclude the technical breakdown, it is helpful to see how these modules interact within the canvas. The following screenshot provides a comprehensive overview of the entire automation flow, illustrating the data path from the initial RSS trigger through the various transformation and distribution stages discussed above.

{{< figure
    src="/images/make_scenario.png"
    alt="The full scenario in make.com"
    caption="The scenario is shared on make.com [here](https://eu1.make.com/public/shared-scenario/zVLytx8htAB/integration-rss-linked-in)"
    class="ma0 w-75"
>}}

## Conclusion: Scalability and Future Automations

Building this pipeline is only the first step. Once you have a reliable data flow from your Hugo site to your social channels, the infrastructure is in place to transform a simple technical blog into a fully automated "Creator" ecosystem. By treating your presence as an interconnected stack, you can begin to automate engagement and even monetization.

#### Expanding the Ecosystem
If you are looking to scale, here are a few directions to take your [Make.com](https://www.make.com/en/register?pc=buzzdeee) scenarios next:

* **Automated Engagement (The "Keyword" Trigger):** On platforms like Instagram, you can set up a "Watch Comments" module. If a user comments a specific keyword (e.g., "SKATE" or "CONFIG"), [Make.com](https://www.make.com/en/register?pc=buzzdeee) can automatically send them a Direct Message with the specific link to your blog post or a download. This significantly boosts engagement and conversion rates without you ever lifting a finger.
    
* **The "Sponsor" Shout-out:**
    If you use platforms like **Patreon**, **Ko-fi**, or **Buy Me a Coffee**, you can add a "Watch New Supporters" trigger. When a new tip or subscription comes in, [Make.com](https://www.make.com/en/register?pc=buzzdeee) can instantly fire off a "Thank You" message to a dedicated `#sponsors` channel in your **Discord** or even post a public "Welcome" graphic to your Facebook Page.

* **Automated Content Archiving:**
    Every time a post is successful, you could have the pipeline append the metadata (URL, date, performance stats) to a **Google Sheet** or an **Airtable** base. This gives you a searchable database of your content history for future "Throwback" posts or end-of-year summaries.

* **Cross-Platform Recycling:**
    You could even set a filter so that if a post performs exceptionally well on LinkedIn, it triggers a "re-post" to a different platform like Reddit or a technical forum after a 48-hour delay.

#### Final Thoughts
In the world of Open Source and BSD, we often talk about "the right tool for the job." [Make.com](https://www.make.com/en/register?pc=buzzdeee), when combined with a well-formatted RSS feed and a bit of Cloudinary logic, becomes that tool. It allows you to maintain a professional, multi-channel presence without sacrificing the time you’d rather spend engineering or skating.

The initial "heavy lifting" of the RSS overrides and API handshakes pays dividends every time you hit `git push`. Once your pipeline is solid, the only thing left to do is keep creating.

