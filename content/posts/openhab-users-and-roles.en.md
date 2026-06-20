---
title: "Securing the Smart Home: Implementing Custom Parent & Child Roles in openHAB"
subtitle: "A practical workaround for tailored parent and child visibility restrictions"
date: 2026-06-20T11:30:25+02:00
lastmod: 2026-06-20T11:30:25+02:00
draft: false
description: "A deep dive into bypassing openHAB's native role limitations to create custom 'parent' and 'child' access controls for your UI layout and semantic model."
license: "MIT"
images: [ "/images/openhab-users-and-roles.png", /images/openhab-default-list-item-visibility-options.png ]

tags: ["openHAB", "Smart Home", "Security"]
categories: ["Smart Home", "Tutorials"]

featuredImage: "/images/openhab-users-and-roles.png"
featuredImagePreview: "/images/openhab-users-and-roles.png"

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
  images: [ "/images/openhab-users-and-roles.png", /images/openhab-default-list-item-visibility-options.png ]
  # ...
---

My home automation setup runs entirely on OpenBSD, providing a rock-solid, secure, and stable foundation for managing the household. However, even on a secure operating system, managing internal family logistics can be a challenge—especially when your kids discover they can turn off the living room TV from their tablets or accidentally fiddle with the master thermostat settings. 

As my own smart home slowly expands, I face a clear necessity: **I needed to hide certain equipment and sensitive controls from my children while keeping full visibility for the adults.**

This sounds like a standard feature for any multi-user system, but implementing it in openHAB turned out to be a fascinating journey through backend databases, Karaf console limitations, and invaluable community wisdom. Here is the story of how I found my way around these structural limitations to finally achieve the setup I was looking for, along with a step-by-step guide so you can do the same.

Feel free to check out [/en/categories/smart-home/](/en/categories/smart-home/) for my other related articles about [openHAB](https://www.openhab.org/) on [OpenBSD](https://www.openbsd.org/).
---

## The Goal: Context-Aware Visibility

The objective was straightforward. I wanted to segregate my household into two primary groups:
* **Parents:** Unrestricted visibility of all dashboards, equipment cards, and advanced metrics (like battery states).
* **Children:** A clean, simplified layout showing only what they need to see, completely hiding sensitive elements.

---

## The Roadblocks & Discoveries

My initial research hit a wall. In the openHAB core architecture, the framework strictly recognizes only two native functional roles for system security:

* **`administrator`**: Granted full, unrestricted access to everything, including system settings, things, items, rules, Developer Tools, and the Add-on Store.
* **`user`**: Granted standard read/write access to basic items, custom pages, and semantic tabs (Overview, Locations, Devices, Properties), while completely hiding the settings panel from the sidebar.

If you try to create a user with a custom role directly, openHAB rejects or ignores it. Furthermore, managing custom roles via the Karaf console is still an [open enhancement request (Issue #2453)](https://github.com/openhab/openhab-core/issues/2453). 

Checking the Karaf console helper output confirmed these rigid boundaries:

{{< highlight bash >}}
openhab> openhab:users ?
Usage: openhab:users list - lists all users
Usage: openhab:users add <userId> <password> <role> - adds a new user with the specified role
Usage: openhab:users remove <userId> - removes the given user
Usage: openhab:users changePassword <userId> <newPassword> - changes the password of a user
{{< /highlight >}}

---

## The Breakthrough: Direct JSONDB Manipulation

To bypass the Karaf console restriction, I realized I had to manually introduce stacked permissions directly into openHAB's internal storage layer. 

By editing the raw JSON database file, you can assign an array of *multiple* roles to a single profile. A user **must** retain either `user` or `administrator` as their foundational role so the Main UI architecture renders properly, but you can append your custom tags right next to it.

Here is the exact procedure:

1. Connect to your openHAB host via SSH and stop the service entirely to prevent memory overwrites:
{{< highlight bash >}}
doas rcctl stop openhab
{{< /highlight >}}

2. Open the users database file in a text editor:
{{< highlight bash >}}
vi /var/db/openhab/jsondb/users.json
{{< /highlight >}}

3. Locate your target user profiles and edit the `"roles"` array to stack your custom strings alongside the core system roles:

{{< highlight json >}}
"my_parent_admin_user": {
  "class": "org.openhab.core.auth.ManagedUser",
  "value": {
    "name": "admin",
    "passwordHash": "...",
    "roles": [
      "administrator",
      "parent"
    ]
  }
},
"other_parent_normal_user": {
  "class": "org.openhab.core.auth.ManagedUser",
  "value": {
    "name": "other_parent",
    "passwordHash": "...",
    "roles": [
      "user",
      "parent"
    ]
  }
},
"one_of_the_childs": {
  "class": "org.openhab.core.auth.ManagedUser",
  "value": {
    "name": "child1",
    "passwordHash": "...",
    "roles": [
      "user",
      "child"
    ]
  }
}
{{< /highlight >}}

4. Save the file and restart openHAB:
{{< highlight bash >}}
doas rcctl start openhab
{{< /highlight >}}

Now, checking the active registry via the Karaf console rewards you with native acknowledgment of your multi-role setup:

{{< highlight bash >}}
openhab> openhab:users list
admin (administrator, parent)
child1 (child, user)
other_parent (parent, user)
{{< /highlight >}}

---

## Community Magic: Applying Visibility to the Semantic Model

With the backend roles configured, I needed a clean way to apply them to the automatically generated semantic pages (Overview, Locations, Equipment). I reached out to the openHAB forum for input on [Custom roles and permissions](https://community.openhab.org/t/custom-roles-and-permissions/169573). 

The community provided the missing piece of the puzzle: customizing the **"Default List Item Widget" metadata** directly on individual items within the Semantic Model tree.

When building layout pages, you can apply visibility controls in two elegant ways.

### Method A: Explicit Role Arrays (`visibleTo`)
The core layout engine respects structural array flags flawlessly. You can explicitly hide standard UI components like an equipment card from the kids:

{{< highlight yaml >}}
- component: oh-equipment-card
  config:
    title: Parent's Bedroom TV
    item: MasterTV_Power
    visibleTo:
      - role:administrator
      - role:parent
{{< /highlight >}}

### Method B: Inline Expressions (`visible`)
Alternatively, if you prefer using evaluating expressions within your custom grid columns, you can tap into the JavaScript-based context engine directly:

{{< highlight yaml >}}
- component: oh-label-card
  config:
    action: navigate
    actionPage: page:BatteryStates
    icon: battery
    title: Battery States
    visible: '=user.roles.includes("parent")'
{{< /highlight >}}

{{< admonition type="tip" title="UI Configuration vs. Code Tab" open=true >}}
There is an important practical difference when building these in the Main UI:
* **Method A (`visibleTo`)**, on the other hand, is limited in the visual UI dropdown selector, which only lets you check the boxes for the native `Administrator` or `User` roles. To inject your custom `parent` or `child` roles here, you must switch over to the widget's **Code** tab and add them manually to the YAML.
* **Method B (`visible`)** can be entered directly into the standard visual editor. You simply enter or paste the string the string `=user.roles.includes("parent")` into the Visibility text field in the UI or paste the string `=user.roles.includes("parent")`.
{{< /admonition >}}

{{< figure
    src="/images/openhab-default-list-item-visibility-options.png"
    alt="List item visibility options"
    caption="List item visibility options"
    class="ma0 w-75"
>}}

Both configurations will cleanly hide target cards from `child1` while keeping them entirely accessible when logged in as `admin` or `other_parent`.

---

## ⚠️ CRITICAL STEP: Close the Backdoor!

You can build the most secure layout rules imaginable, but they are completely useless if openHAB leaves the front door unlocked. By default, openHAB ships with an anonymous fallback option for seamless browser connectivity.

If a child opens the Main UI dashboard without logging in, openHAB assigns them an **Implicit User Role**. They will immediately bypass your custom roles and view every single piece of hidden equipment!

{{< admonition type="warning" title="Security Requirement" open=true >}}
You **must** turn off anonymous fallbacks immediately after setting up your users:
1. Open the Main UI as an Administrator.
2. Navigate to **Settings** -> **API Security**.
3. Toggle **Implicit User Role** to **OFF**.
{{< /admonition >}}

With this switched off, any unauthenticated device will be met with a strict login prompt, forcing them to authenticate into their restrictive `child` profile before seeing any interfaces.

---

## 🛑 Important Caveat: Visibility is NOT Absolute Security

As highlighted in the official openHAB UI documentation and in the screenshot above, **visibility restrictions are strictly a user interface convenience feature, not a concrete security boundary.**

When you use properties like `visibleTo` or `visible`, the openHAB Main UI simply stops rendering that component inside the web browser or mobile app. However, the backend items still technically exist and listen for commands. If a tech-savvy child or guest knows how to inspect network requests or interact directly with the openHAB REST API, they can still view state changes and issue commands to hidden hardware.

For strict, absolute hardware isolation, you must pair these visibility filters with rigid server-side execution permissions, scripts, or dedicated secondary instances if your network security requirements are absolute.

---

## Conclusion

While openHAB doesn't natively expose custom roles inside its visual administration menus yet, combining stacked role arrays in the `jsondb` with standard widget visibility expressions works flawlessly. It bridges the gap between powerful, unfiltered administrative utility and a safe, child-friendly smart home interface.

Interestingly, raising this topic in the forum triggered a fascinating, larger debate within the community regarding Role-Based Access Control (RBAC) in home automation architectures. Finding the right balance between slick UI element masking and hard backend API security boundaries is a hot topic, and it is great to see the community actively brainstorming future improvements.

A huge thank you to the openHAB community for pointing me toward list widget metadata mapping! Hopefully, this guide helps you secure your own control panels. Happy automating!

