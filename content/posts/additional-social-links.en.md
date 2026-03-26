---
title: "Adding Custom Social Icons to Hugo LoveIt"
subtitle: "How to seamlessly integrate Ko-fi, Liberapay, and other platforms into your theme"
date: 2026-03-26T16:06:36+01:00
lastmod: 2026-03-26T16:06:36+01:00
draft: false
description: "Learn how to extend the Hugo LoveIt theme to add social media icons like Ko-fi or Liberapay that are not supported out of the box."
license: "MIT"
images: []

tags: [ "Hugo", "LoveIt", "Web Design", "Tutorial", "FontAwesome" ]
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
share:
  enable: true
comment:
  enable: false
---

## Why bother?

The Hugo theme **LoveIt** offers a fantastic out-of-the-box solution for social media icons. But what happens if you use platforms that aren't included in the standard repertoire? If you fund your work via **Ko-fi**, **Liberapay**, or other specialized networks, you'll naturally want to link them prominently in your sidebar or footer.

Since LoveIt is built on Font Awesome and a solid SVG library, the solution is already there - we just need to teach the theme how to recognize the new links and assign the correct icons.

## Implementation

To add a new icon, we don't need to dig through the theme's core code. Instead, we use Hugo's **overwrite** principle.

### Configuration (hugo.toml / config.toml)

First, add your link as usual under `[params.social]`. LoveIt searches for the icon based on the name (key) you provide, so we choose identifiers that we will later define in our data file.

{{< highlight toml "linenos=table,open=false" >}}
[params.social]
**These names must match the keys in your social.yml**

kofi = "your-username"
liberapay = "your-username"
{{< /highlight >}}

### Implementation via `social.yml`

Instead of messing with HTML templates, LoveIt uses a central definition file for all social media services. By default, this is located at `assets/data/social.yml`.

#### Overriding `social.yml`

First, copy the file into your local project directory to ensure your changes aren't lost during a theme update:
`themes/LoveIt/assets/data/social.yml` -> `assets/data/social.yml`

#### Registering New Services

Open the copied file and add your new services at the end (or at your preferred position). **LoveIt** distinguishes between **Font Awesome** icons and **Simple Icons** (SVG).

{{< highlight yaml "linenos=table,open=false" >}}
# 082: Ko-Fi (Uses FontAwesome Brands)
kofi:
  Weight: 39
  Title: Ko-Fi
  Prefix: https://ko-fi.com/
  Icon:
    Class: fa-brands fa-ko-fi

# 083: Liberapay (Uses the integrated SVG library)
liberapay:
  Weight: 37
  Title: Liberapay
  Prefix: https://liberapay.com/
  Icon:
    Simpleicons: liberapay
{{< /highlight >}}

## 🔍 How to find the correct icon names?

To ensure the icon displays correctly, you need to know its exact class name or identifier. Depending on your source, here is how you proceed:

1. **For Font Awesome (Class)** Visit the official [Font Awesome Icon Search](https://fontawesome.com/icons).
   * Search for your platform (e.g., "Ko-fi").
   * Make sure to set the "**Free**" filter.
   * Copy the class name. In your `social.yml`, you usually need to prepend the prefix for brands (`fa-brands`) or solid icons (`fa-solid`).

       * Example: `fa-brands fa-ko-fi`

2. **For Simple Icons (SVG)** If an icon is missing from Font Awesome (common for open-source projects), check your theme's local directory: `./themes/LoveIt/assets/lib/simple-icons/icons/`
   * The name of the `.svg` file (without the extension) is your key for the `social.yml`.
   * **Important**: The name must match **exactly** (usually all lowercase). If the file is named `liberapay.svg`, enter `liberapay`.

### The Wildcard: Integrating Your Own SVG Icons

What if a brand-new platform or a very niche service is found neither in Font Awesome nor in the Simple Icons library? No problem - you can manually add any SVG icon you like.

#### Step 1: Saving the Icon

Obtain the logo as an `.svg` file (many brands provide these in their press kits). Save the file in your local project folder at: `assets/lib/simple-icons/icons/my-new-service.svg`

{{< admonition type="info" open=true >}}
**Important**: Recreate the folder structure in your root directory (`assets/...`) if it doesn't exist yet. Hugo prioritizes your local files over those in the themes folder.
{{< /admonition >}}

{{< admonition type="info" open=true >}}
If you only have a high-resolution PNG or JPG of the logo, you can have it vectorized automatically.

* **Tools**: Use free online converters like Vector Magic or the Adobe Express Logo Converter.
* **Process**: Upload image → Start vectorization → Export as `.svg`.
* **Important**: Ensure the background is transparent; otherwise, you'll end up with a white box around your icon.
{{< /admonition >}}

#### Step 2: Registering in social.yml

Now you can reference your custom icon just like a standard one. The name under Simpleicons must exactly match the filename you just created:

{{< highlight yaml "linenos=table,open=false" >}}
# 084: My New Service
my-new-service:
  Weight: 40
  Title: My Service
  Prefix: https://example.com/
  Icon:
    Simpleicons: my-new-service
{{< /highlight >}}

{{< figure
    src="/images/custom_social_links.png"
    alt="Custom Social Links"
    caption="Custom Social Links"
    class="ma0 w-75"
>}}

## Conclusion

With Hugo LoveIt, you are never restricted to the default presets. Whether you use Font Awesome, the massive Simple Icons library, or even your own custom SVG files—you have full control over how and where you showcase your profiles. By mirroring the social.yml into your local directory, your setup remains clean, update-proof, and easily expandable at any time.

Happy networking!
