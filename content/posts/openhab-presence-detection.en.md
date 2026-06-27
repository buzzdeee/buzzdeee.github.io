---
title: "Project NCC-1701: Smart Home Presence Detection with openHAB and UniFi"
subtitle: "How a sensor-based bridge phalanx tames the openHAB engine room."
date: 2026-06-27T10:49:41+02:00
lastmod: 2026-06-27T10:49:41+02:00
draft: false
description: "A guide to stable presence detection using the openHAB Network and UniFi bindings. Bypass the UI Equipment pitfall and bring your crew to the dashboard using the At Home Card."
license: "MIT"
images: [ "/images/Crew_On_Deck.png", "/images/pingable_device.png", "/images/unifi_client.png", "/images/Group_kirk.png" ]

tags: ["StarTrek", "openHAB", "SmartHome", "UniFi", "PresenceDetection"]
categories: [ "Smart Home" ]

featuredImage: "/images/Crew_On_Deck.png"
featuredImagePreview: "/images/Crew_On_Deck.png"

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
  images: [ "/images/Crew_On_Deck.png", "/images/pingable_device.png", "/images/unifi_client.png", "/images/Group_kirk.png" ]
---


# "All hands on deck, Scotty?" 

Anyone familiar with [openHAB](https://www.openhab.org/) knows that the learning curve is steep, but the possibilities are incredibly vast. You quickly realize that rigid schedules don't make a house truly *smart*. A real smart home needs to know who is on board to intelligently manage life support systems (heating, lighting, security).

As a long-time Trekkie, the design for my dashboard was a no-brainer: the classic **TOS (The Original Series) crew** would represent our family of five. When a crew member is on board, their avatar lights up in full color on the UI widget. If they leave the ship, the display switches to *grayscale*, signaling their "Away Mission" status. 

What sounds simple on paper quickly felt like someone brought a box of Tribbles into the openHAB engine room: nothing worked right "out of the box". With a bit of patience, some trial and error while searching for the correct options, and fine-tuning the parameters, the communicator phalanx was eventually calibrated successfully. Since I am still relatively at the beginning of my smart home odyssey, I wanted to share my experiences with IP and Wi-Fi tracking alongside proper group configuration.

Feel free to check out [/en/categories/smart-home/](/en/categories/smart-home/) for my other related articles about [openHAB](https://www.openhab.org/) on [OpenBSD](https://www.openbsd.org/).

---

## 1. Preparing the Communicators (Android & iOS)

Before openHAB can track our mobile devices, the smartphones need to be configured to play nice with the local network. Modern operating systems are optimized for maximum privacy, which introduces two default features that break persistent Wi-Fi tracking.

### iOS (iPhone)
* **Disable Private Wi-Fi Address:** By default, Apple randomizes its MAC address every time it connects. This breaks any attempts at stable network tracking.
  * *Path:* `Settings` -> `Wi-Fi` -> Tap the blue **(i)** next to your home network -> **Turn off Private Wi-Fi Address**.
* **Static IP Address Sourcing:** Make sure your router (e.g., OpenBSD, UniFi, or FritzBox) is configured via DHCP reservation to always assign the same IP address to the iPhone.

{{< admonition type="note" title="Deep Dive: Comparing Apple's MAC Address Modes" open=true >}}
In recent iOS versions, Apple differentiates between three states for your Wi-Fi MAC address:

1. **Rotating:** The MAC address changes periodically, even on the same network. This completely breaks local presence tracking.
2. **Fixed (Private):** The iPhone uses a randomized MAC address, but it stays *consistent for this specific network*.
3. **Off:** The iPhone transmits its true, hardware-burned MAC address to the router.

#### Pros & Cons in a Home Network: "Off" vs. "Fixed"

| Mode | Pros | Cons |
| :--- | :--- | :--- |
| **Off** | • Absolute binding stability.<br>• Ideal for strict MAC filters on the router.<br>• Unaffected by major iOS updates. | • Disables tracking protection entirely for this specific SSID.<br>• One less layer of privacy inside your own LAN. |
| **Fixed** | • Good privacy compromise (the router doesn't see your true hardware MAC) while allowing static tracking. | • If you ever select "Reset Network Settings" on your iPhone, a new fixed MAC is generated, breaking the openHAB link. |

**Recommendation for your Smart Home:** Since your home network is a trusted zone, turning the feature completely **Off** is the most hassle-free option for the Captain. If you prefer maximum data minimization, go with **Fixed**, but remember to reconfigure the client profile (MAC address) if the device is ever network-reset.
{{< /admonition >}}

### Android
* **Disable Randomized MAC Address:** Google also uses randomized addresses by default.
  * *Path:* `Settings` -> `Network & internet` -> `Internet` -> Tap the gear icon next to your home Wi-Fi -> `Privacy` -> Select **Use device MAC**.

---

## 2. The Bridge Crew: Who's Who?

To model our family cleanly into the openHAB Semantic Model, I mapped everyone to a member of the iconic bridge crew:

* **Dad:** `Captain Kirk` (Maintains tactical overview)
* **Mom:** `Uhura` (Keeps the family's communication channels running smoothly)
* **Child 1:** `Spock` (The logical mind, permanently glued to a screen)
* **Child 2:** `Scotty` (Our raw energy core in the engine room)
* **Child 3:** `Bones / McCoy` (The youngest, always ready to mutiny)

---

## 3. Local Tracking: IP-Ping vs. UniFi WiFi Client

I tested two distinct methods for tracking devices within the local network area.

### Method A: The "Pingable Network Device" (Network Binding)
This method relies on the standard Network Binding, which should be installed beforehand. openHAB regularly fires a brief network ping to the communicator's static IP address. If the device replies, its status switches to `ON`.

#### Configuration Code for the Pingable Device
Here is an example for Captain Kirk; the other crew members follow the identical structure:
{{< highlight yaml >}}
Thing network:pingdevice:KirkCommunicator "Kirk Communicator Ping" [hostname="192.168.0.42", macAddress="11:22:33:44:55:66", refreshInterval=60000, retry=1, timeout=5000, useIOSWakeUp=true, useArpPing=true, useIcmpPing=true] {
        Channels:
                Type online : online "Online"
                Type latency : latency "Latency"
                Type lastseen : lastseen "Last Seen"
}
{{< /highlight >}}

* **The Verdict:** Fast setup that works with any generic router. **The Problem:** When smartphones (especially iPhones) enter a deep power-saving sleep state, they drop their active Wi-Fi connection temporarily. openHAB assumes they left the ship, causing frequent false alarms.

{{< figure
    src="/images/pingable_device.png"
    alt="24-hour connectivity chart of the Pingable Device"
    caption="24-hour connectivity chart of the Pingable Device"
    class="ma0 w-75"
>}}


### Method B: The UniFi Wi-Fi Client Device (UniFi Binding)
This method utilizes the specialized UniFi Binding. Since my home network is powered by a Ubiquiti UniFi stack, this turned out to be the significantly more robust alternative. openHAB directly queries the UniFi controller to verify if the specific **MAC address** of the communicator is registered on the wireless network.

#### Configuration Code for the UniFi Client
An example for Captain Kirk; replicate equivalently for the rest of your crew:
{{< highlight yaml>}}
Thing unifi:wirelessClient:KirkCommunicator "Kirks Communicator" (unifi:controller:e5b4544ee2) [cid="11:22:33:44:55:66", considerHome=180] {
        Channels:
                Type online : online "Online"
                Type name : name "Name"
                Type hostname : hostname "Hostname"
                Type site : site "Site Name"
                Type macAddress : macAddress "MAC Address"
                Type ipAddress : ipAddress "IP Address"
                Type uptime : uptime "Uptime"
                Type lastSeen : lastSeen "Last Seen"
                Type blocked : blocked "Blocked"
                Type experience : experience "Experience"
                Type guest : guest "Guest"
                Type ap : ap "Access Point"
                Type essid : essid "Wireless Network"
                Type rssi : rssi "Received Signal Strength Indicator"
                Type wirelessCmd : cmd "Wireless Command"
                Type reconnect : reconnect "Reconnect"
}
{{< /highlight >}}

* **The Verdict:** A massive upgrade. An iPhone registers with the UniFi access points as you pull up to the house, even *before* it gets assigned an IP address via DHCP. The controller tracks the client even when it sleeps. As an added bonus, the binding exposes exactly which access point (e.g., in the engine room or Forward-and-Ten) the crew member is closest to.

{{< figure
    src="/images/unifi_client.png"
    alt="24-hour connectivity chart of the Wi-Fi client"
    caption="24-hour connectivity chart of the Wi-Fi client"
    class="ma0 w-75"
>}}

Much better!

---

## 4. The openHAB Riddle: Group & Item Architecture

This is where I ran into the biggest engineering bottleneck. I had initialized an "Equipment" group for each crew member. Inside that group sat individual tracking items for their communicator's state (Ping and UniFi Client status) alongside a string item showing their active Access Point connection.

Even though the underlying items accurately registered as `Online`, the parent Equipment groups remained stubbornly stuck in a `NULL` (uninitialized) state, leaving the dashboard widget grayed out. Why? By default, a standard openHAB group has no idea how to aggregate the states of its children. You have to explicitly instruct the group to evaluate the state of its nested `Switch` items using a logical function.

The structural blueprint of our groups and items looks like this:

{{< highlight text >}}
Group:Switch:OR(ON, OFF) Crew "Crew" <f7:person_3>

Group:Switch:OR(ON, OFF) Kirk "Kirk" <man_3> (Crew) ["Equipment"]
Group:Switch:OR(ON, OFF) Uhura "Uhura" <woman_3> (Crew) ["Equipment"]
Group:Switch:OR(ON, OFF) Spock "Spock" <boy_3> (Crew) ["Equipment"]
Group:Switch:OR(ON, OFF) Scotty "Scotty" <boy_4> (Crew) ["Equipment"]
Group:Switch:OR(ON, OFF) McCoy "McCoy" <boy_5> (Crew) ["Equipment"]
{{< /highlight >}}

{{< admonition type="warning" title="The UI Pitfall: Equipment vs. Switch Groups" open=true >}}
This reveals a core limitation within openHAB's MainUI web editor: 
* If you create a `Group` type item and apply the semantic tag **Equipment**, the UI hides the option to assign an aggregation function.
* However, if you switch the group type to **Switch** inside the UI to unlock aggregation, the UI will often refuse to let you nest child elements correctly within your semantic model.

**The Fix:** Leave the group type configured as an *Equipment* in the visual UI, then navigate over to the **Code Tab (YAML)** to inject the required `group` logic block manually.

**Note on Aggregation Behavior:** Because the group is explicitly configured with `type: Switch`, openHAB's aggregation engine will only evaluate nested items of the `Switch` type (like the Ping and UniFi online states). It automatically ignores all other types—such as the `String` type used for the `Kirk_Subspace_Node`—preventing format conflicts and keeps the tracking logic clean.

**Why OR(ON, OFF) instead of AND(ON, OFF)?**
While an `AND` chain would require *both* the Ping and the UniFi binding to report `ON` at the same time, presence tracking only needs a logical `OR`. This means that as long as at least one sensor reports the communicator is within range, the crew member is considered on board. Only when both channels independently drop to `OFF` does the entire equipment group switch to absent. This perfectly buffers temporary dropouts, such as the iPhone's deep sleep state during pings.

**The Crew Group: Why OR is the most useful choice here as well**
For the master `Crew` group, we also implement an `OR(ON, OFF)` aggregation. This comes down to a simple, practical automation rule: the group is meant to reflect a global "Someone is home" status (e.g., to disarm the security grid the moment the first person steps through the door). Using an `AND` function here would be useless, as the group would only trip to `ON` if every single family member was on board simultaneously—meaning the system would falsely treat the house as empty even if four out of five people were sitting on the bridge.
{{< /admonition >}}

### YAML Code for the "Kirk" Equipment Group
{{< highlight yaml >}}
version: 1
items:
  Kirk:
    type: Group
    label: Kirk
    icon: man_3
    groups:
      - Crew
    tags:
      - Equipment
    group:
      type: Switch
      function: OR
      parameters:
        - ON
        - OFF
{{< /highlight >}}

### DSL Code for Items inside the Kirk Group:
{{< highlight text >}}
Switch Kirk_Online "Communicator Online" <f7:antenna_radiowaves_left_right> (Kirk) ["Presence", "Status"] {
  channel="unifi:wirelessClient:KirkCommunicator:online",
  stateDescription=" " [options="ON=Online,OFF=Offline", readOnly=true]
}
Switch Kirk_Communicator_Ping "Communicator Ping" <f7:waveform_path_ecg> (Kirk) ["Presence", "Status"] {
  channel="network:pingdevice:KirkCommunicator:online",
  stateDescription=" " [options="ON=Online,OFF=Offline", readOnly=true]
}
String Kirk_Subspace_Node "Subspace Node" <f7:wifi> (Kirk) ["Point"] {
  channel="unifi:wirelessClient:KirkCommunicator:ap",
  stateDescription=" " [readOnly=true]
}
... (Replicate this structure identically for other crew members)
{{< /highlight >}}

The working result in the model should look like this:

{{< figure
    src="/images/Group_kirk.png"
    alt="Captain Kirk is on board the Enterprise"
    caption="Captain Kirk is on board the Enterprise"
    class="ma0 w-75"
>}}

---

## 5. Visualizing the Bridge: The "At Home Card"

Now that the logic in the engine room is running smoothly, let's display it on the main viewer. For the user interface, I used the excellent **At Home Card** custom widget available via the openHAB Community Marketplace. 

This widget targets a parent group (our `Crew` group), automatically discovers the nested equipment entries (our crew members), and smoothly toggles their avatars between full color (`ON`) and desaturated grayscale (`OFF`).

### Step 1: Storing Avatars in the File System
To allow the widget to fetch our TOS crew portraits, they need to reside within openHAB's local HTML path.

{{< admonition type="tip" title="Strict Asset Naming Rules" open=true >}}
The widget maps images to elements based on the exact **Item Name** (not its display label!). If your item is named `Kirk`, your asset must be saved precisely as `Kirk.png` (or `.jpg`). The mapping is strictly case-sensitive.
{{< /admonition >}}

Copy your images to your openHAB host environment and place them in the following directory structure:

Path on the server: `/etc/openhab/html/presence/`

Upload the five crew portraits here:
* `Kirk.png`
* `Uhura.png`
* `Spock.png`
* `Scotty.png`
* `McCoy.png`

These assets are now exposed locally via openHAB's web server at `http://<your-openhab-ip>:8080/static/presence/Kirk.png`.

### Step 2: Widget Installation & Layout Configuration
1. Open the MainUI and head over to the **Add-on Store**.
2. Find the **At Home Card** under UI Widgets and click **Install**.
3. Create a new custom Page layout or open an existing one.
4. Drop the widget onto your layout canvas, switch into its **Code Tab**, and configure it using this YAML scheme:

{{< highlight yaml >}}
component: widget:at_home_card
config:
  family: Crew
  title: On Deck
  bggradienttopcolor: rgba(253, 242, 201, 1)
  bggradientbottomcolor: rgba(245, 220, 160, 1)
{{< /highlight >}}

The `family: Crew` property points the card directly to our root group. Saving the page reveals your bridge crew cleanly aligned. Anyone currently on board lights up in full uniform color, while those out on a mission remain grayed out.

{{< figure
    src="/images/Crew_On_Deck.png"
    alt="Everyone accounted for?"
    caption="Everyone accounted for?"
    class="ma0 w-75"
>}}

---

## 6. Conclusion & Future Roadmap

Presence mapping via local Wi-Fi states and UniFi tracking gives us a highly reliable foundation. The crew status updates seamlessly across the main bridge consoles.

Keep in mind that wireless connection monitoring is only **one** sensor channel in a complete automation array. It mainly targets immediate proximity tracking. When leaving the geofence area, it naturally takes a few minutes before the UniFi controller safely flags a client device as disconnected.

{{< admonition type="note" title="Upcoming Refinements" open=true >}}
To build a bulletproof presence architecture, I plan to introduce these auxiliary tracking channels:
1. **[GPS Position Streams via OwnTracks](https://www.openhab.org/addons/bindings/gpstracker/):** Feeds positional tracking strings back to openHAB from further out, letting the system spin up heating zones when I cross a 500-meter threshold.
2. **Bluetooth Low Energy (BLE) Beacons:** Lightweight transponders attached to keyfobs tracked by local ESP32 nodes distributed through the house to unlock precise, room-level interior tracking.
{{< /admonition >}}

Our voyage is just beginning, but a vital engineering goal is achieved. The ship finally knows exactly who is on board.

*How do you handle presence tracking in your home automation setup? Do you stick to local Wi-Fi hooks, or do you rely entirely on mobile GPS engines? Let me know in the comments below!*

