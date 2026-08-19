---
title: "Building an OpenBSD Home Automation Gateway with Zigbee2MQTT and openHAB"
subtitle: "tailoring a silent, isolated IoT gateway using Zigbee2MQTT, Mosquitto, and openHAB on fanless x86 hardware"
date: 2026-06-11T16:52:20+02:00
lastmod: 2026-06-11T16:52:20+02:00
draft: false
authorLink: ""
description: "A deep dive into setting up a secure home automation server on a Fujitsu Futro S9010n with OpenBSD. Covers Nginx proxy setup, dealing with GraalVM roadblocks on non-mainstream OS, and automated file-based Thing generation."
license: "MIT"
images: [ "/images/openbsd-home-automation-gateway-with-zigbee2mqtt-and-openhab.png", "edited-image.jpg", "/images/zigbee2mqtt_network.png", "/images/zigbee2mqtt_dashboard.png", "/images/openHAB_addons.png", "/images/openHAB_zigbee_things.png", "/images/openHAB_zigbee_presence_sensor_channels.png", "/images/openHAB-items.png", "/images/openHAB-pages.png", "/images/openHAB-mobile.png" ]

tags: ["OpenBSD", "Zigbee", "Home Automation", "openHAB", "Nginx", "MQTT"]
categories: [ "Smart Home" ]

featuredImage: "/images/openbsd-home-automation-gateway-with-zigbee2mqtt-and-openhab.png"
featuredImagePreview: "/images/openbsd-home-automation-gateway-with-zigbee2mqtt-and-openhab.png"

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
  images: [ "/images/openbsd-home-automation-gateway-with-zigbee2mqtt-and-openhab.png", "edited-image.jpg", "/images/zigbee2mqtt_network.png", "/images/zigbee2mqtt_dashboard.png", "/images/openHAB_addons.png", "/images/openHAB_zigbee_things.png", "/images/openHAB_zigbee_presence_sensor_channels.png", "/images/openHAB-items.png", "/images/openHAB-pages.png", "/images/openHAB-mobile.png" ]
  # ...
---

If you’ve been following my previous adventures in home automation, you know I’ve been sliding down the Zigbee rabbit hole for a while. It all started with getting my hands dirty tracking frames in [Zigbee-Sniffing unter OpenBSD: Deep Dive mit dem TI CC2531](/en/zigbee-sniffing-on-openbsd/). Out of curiosity, I got myself a Zigbee Coordinator and wrote about connecting it to an automation platform in [Zigbee Home Automation auf OpenBSD: Sonoff Dongle-M und openHAB](/en/sonoff-dongle-m-with-openhab-on-openbsd/). 

Recently, I picked up a batch of surplus thin clients that turned out to be the absolute perfect fit for a dedicated home controller. Here is how I built a robust, isolated smart home gateway using OpenBSD 7.9-snapshot, Zigbee2MQTT, Mosquitto, and openHAB.

---

## The Hardware Stack

The foundation of this build is the [Fujitsu Futro S9010n](https://www.ebay.de/sch/i.html?_nkw=Fujitsu+Futro+S9010n&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=FutroS9010n&toolid=10001&mkevt=1#affiliate). These little machines are absolute gems for home servers. For under the price of a modern Raspberry Pi kit, you get a fully enclosed x86 machine equipped with:

* **20 GB RAM** (Originally came with 8GB, but was able to make it work with 20GB, 32GB made it slow like hell)
* **64 GB SSD**
* **Fanless cooling** (totally silent in a living room or utility closet)
* **Dual native serial ports**

{{< admonition type="warning" title="OpenBSD Graphics Quirks" open=true >}}
There is one specific caveat if you run OpenBSD on the Futro S9010n: the display goes completely blank if `inteldrm` is enabled. Since this is run completely headless as a server, it doesn't bother me, but keep it in mind during initial installation!
{{< /admonition >}}

To keep things secure, I locked this server into a dedicated IoT VLAN. Network communication out to the physical Zigbee mesh is handled over the network via a [SONOFF Zigbee 3.0 USB Dongle Plus (Dongle-M)](https://www.ebay.de/itm/388971238285?mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=DongleM&toolid=10001&mkevt=1#affiliate), isolating untrusted endpoints entirely from my primary network lanes.

With the backbone ready, I loaded up an array of smart devices to manage:
* **Power:** A [4-Pack of SONOFF Zigbee Smart Plugs](https://www.ebay.de/itm/385779868557?mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=SonoffSteckdose&toolid=10001&mkevt=1#affiliate) (great for mapping routers across the mesh).
* **Climate:** A [SONOFF Smart Zigbee 3.0 Thermostat](https://www.ebay.de/itm/388766053368?mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Thermostat&toolid=10001&mkevt=1#affiliate) as well as [SONOFF SNZB-02P Temperature & Humidity Sensors](https://www.ebay.de/itm/384895576208?mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=TempFeuchtSensor&toolid=10001&mkevt=1#affiliate).
* **Presence & Lighting:** The microwave-radar-based [SONOFF SNZB-06P Presence Sensor](https://www.ebay.de/itm/386285050110?mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=PresenceSensor&toolid=10001&mkevt=1#affiliate) driving standard [GU10 Smart Zigbee LED Bulbs](https://www.ebay.de/sch/i.html?_nkw=Smart+WiFi+Zigbee+LED+Glühbirne+GU10&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Gluehbirne&toolid=10001&mkevt=1#affiliate) and a matching [Zigbee Deckenlampe](https://www.ebay.de/itm/326908849090?mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Deckenlampe&toolid=10001&mkevt=1#affiliate).

---

## Software Configuration

The architecture is layered sequentially: Nginx intercepts external traffic, authenticates requests, and reverse-proxies them downward to openHAB or Zigbee2MQTT, which interact using the Mosquitto MQTT broker. The base configurations are deployed via OpenVOX, with simple daily shell scripts handling automated state backups.

### 1. Zigbee2MQTT
The core Zigbee translation engine runs on port `8080`. Note that we are passing the adapter connection via TCP to reach the isolated network segment where the Sonoff Dongle-M resides.

{{< highlight yaml >}}
version: 5
homeassistant:
  enabled: true
  experimental_event_entities: true
  legacy_action_sensor: true
frontend:
  enabled: true
  host: 127.0.0.1
  base_url: /zigbee2mqtt
  port: 8080
mqtt:
  base_topic: zigbee2mqtt
  server: mqtt://localhost
  version: 5
serial:
  port: tcp://sonoff.example.com:6638
  baudrate: 115200
  adapter: ember
  rtscts: false
advanced:
  log_level: info
  log_directory: /var/log/zigbee2mqtt
  log_file: zigbee2mqtt_%TIMESTAMP%.log
  log_rotation: true
  log_output:
    - file
  network_key:
    # ... secret key ...
{{< /highlight >}}

{{< admonition type="note" title="Configuration Management Pain" open=true >}}
Managing this specific file with automation tools like OpenVox or Ansible can be a total pain. Because the Zigbee2MQTT daemon actively modifies this file dynamically when editing states or pairing new hardware, it easily conflicts with declarative configuration management files.
{{< /admonition >}}

### 2. Mosquitto & openHAB 5.1.3
Mosquitto requires zero wizardry; pull the package from the OpenBSD repositories, enable it via `rcctl`, and let it run. 

For openHAB, installation is similarly straightforward. To prevent it from clashing with the Zigbee2MQTT frontend port, remember to map its listening port out of the box to `8081` within `/etc/openhab.conf`:

{{< highlight bash >}}
OPENHAB_HTTP_PORT=8081
{{< /highlight >}}

### 3. Nginx Reverse Proxy
By routing traffic through Nginx, we can run openHAB securely at the root directory `/` while containing the raw Zigbee2MQTT administration panel safely behind an explicit HTTP basic authentication barrier at `/zigbee2mqtt`.

{{< highlight nginx >}}
# MANAGED BY OpenHAB
server {
  listen       *:443 ssl;
  listen       [::]:443 ssl ipv6only=on;

  server_name  ha ha.example.com;

  http2 off;
  ssl_certificate      /etc/puppetlabs/puppet/ssl/certs/ha.example.com.pem;
  ssl_certificate_key  /etc/puppetlabs/puppet/ssl/private_keys/ha.example.com.pem;

  index  index.html index.htm index.php;
  access_log            /var/www/logs/ssl-ha.example.com.access.log;
  error_log             /var/www/logs/ssl-ha.example.com.error.log;

  location /zigbee2mqtt {
    auth_basic           "Home Automation Area";
    auth_basic_user_file /conf/.htpasswd;
    proxy_pass            http://127.0.0.1:8080;
    proxy_read_timeout    90s;
    proxy_send_timeout    90s;
    proxy_http_version    1.1;
    proxy_set_header      Host $host;
    proxy_set_header      X-Real-IP $remote_addr;
    proxy_set_header      X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header      X-Forwarded-Host $host;
    proxy_set_header      X-Forwarded-Proto $scheme;
    proxy_set_header      Proxy "";
    client_body_buffer_size 128k;
    client_max_body_size 8m;
    proxy_buffers 4 32k;
    proxy_redirect http:// https://;
    proxy_set_header Connection "upgrade";
    proxy_set_header Upgrade $http_upgrade;
  }

  location / {
    proxy_pass            http://127.0.0.1:8081;
    proxy_read_timeout    90s;
    proxy_send_timeout    90s;
    proxy_http_version    1.1;
    proxy_set_header      Host $host;
    proxy_set_header      X-Real-IP $remote_addr;
    proxy_set_header      X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header      X-Forwarded-Host $host;
    proxy_set_header      X-Forwarded-Proto $scheme;
    proxy_set_header      Proxy "";
    client_body_buffer_size 128k;
    client_max_body_size 8m;
    proxy_buffers 4 32k;
    proxy_redirect http:// https://;
    proxy_set_header Connection "upgrade";
    proxy_set_header Content-Type $http_content_type;
    proxy_set_header Upgrade $http_upgrade;
  }
}
{{< /highlight >}}

---

## Device Pairing & The OpenBSD Roadblock

Once Nginx was reloaded, pairing endpoints inside the Zigbee2MQTT dashboard worked flawlessly. You toggle joining mode via the UI, press the pairing buttons on the physical sensors, and they instantly populate the interface. The mapping engine compiles a gorgeous visualization of the entire interconnected hardware mesh 

{{< figure
    src="/images/zigbee2mqtt_network.png"
    alt="Zigbee network mesh"
    caption="Zigbee network mesh"
    class="ma0 w-75"
>}}

alongside live variable readouts 

{{< figure
    src="/images/zigbee2mqtt_dashboard.png"
    alt="Zigbee devices dashboard"
    caption="Zigbee devices dashboard"
    class="ma0 w-75"
>}}

However, passing those recognized devices over to openHAB hit a massive structural roadblock.

### The Home Assistant Binding Exception
Typically, based on my research, you would simply spin up the **Home Assistant Binding** inside openHAB to automatically ingest and parse the MQTT discovery topics broadcasted by Zigbee2MQTT. That would magically populate your Things with their Channels. Unfortunately, under openHAB 5.1, this binding relies on internal native components via GraalVM that crash out on non-mainstream operating systems like OpenBSD.

I eventually reached out and asked about the issue on the [openHAB Community Forums](https://community.openhab.org/t/home-assistant-binding-exception-on-openbsd/169485). The experience was awesome—the community was incredibly quick to respond with highly helpful insight, pointing me directly to the core tracking issue on [GitHub (#19276)](https://github.com/openhab/openhab-addons/issues/19276).

{{< admonition type="info" title="Light at the End of the Tunnel" open=true >}}
GraalVM devs are adding an explicit configuration flag in their upcoming tracking milestone to restore native support across non-mainstream operating systems. While it won't make it in time for openHAB 5.2, we should see official resolution by version 5.3!
{{< /admonition >}}

### The Workaround: A Custom Discovery Bridge
Since I didn't want to map dozens of channels manually in the openHAB UI, I wrote a lightweight python helper script that runs as a system daemon. It listens directly to the Mosquitto MQTT broker configuration topics and dynamically writes out matching `.things` files into `/etc/openhab/things/`. 

It is still explicitly fitted to the properties of my specific collection of switches and sensors, but it saves hours of tedious UI assembly. You can grab or fork the script here: [github.com/buzzdeee/mqtt_discovery_bridge](https://github.com/buzzdeee/mqtt_discovery_bridge).

### Inhabiting openHAB: Extensions and Populated Things

With the bridge script automatically managing file-based generation on the backend, we still need to make sure openHAB has the core translation tools installed to understand the incoming data stream. 

Heading into the openHAB Add-on Store, I installed the foundational **MQTT Binding** alongside a few essential data transformations required to handle the payload varieties: **Jinja**, **JSONPath**, and **Regex**.

{{< figure
    src="/images/openHAB_addons.png"
    alt="Installed openHAB Add-ons showing MQTT and transformations"
    caption="The required MQTT binding and transformation add-ons inside openHAB"
    class="ma0 w-75"
>}}

Once those add-ons were active and the Python daemon started spitting out files into the configuration directory, openHAB automatically picked them up. Navigating over to **Settings -> Things** reveals the payoff: the entire roster of Zigbee devices is populated, initialized, and online without clicking through a single wizard loop.

{{< figure
    src="/images/openHAB_zigbee_things.png"
    alt="openHAB Things list showing all populated Zigbee devices online"
    caption="The automatically populated list of Zigbee hardware Things"
    class="ma0 w-75"
>}}

Opening up a specific entity—in this case, the microwave-radar-based SONOFF presence sensor—shows how cleanly the properties are mapped out. The internal state exposes its available operational data pipelines, giving me instant access to properties like presence detection, illumination metrics, and motion status channels ready to link to UI items or automation scripts.

{{< figure
    src="/images/openHAB_zigbee_presence_sensor_channels.png"
    alt="Detailed channel layout of the SONOFF presence sensor in openHAB"
    caption="Inspecting the generated data channels of the SONOFF Presence Sensor"
    class="ma0 w-75"
>}}

### Linking Items, Building Pages, and Going Mobile

With the data channels exposed, the final step to make this fully interactive is creating human-readable **Items** and pinning them to a control dashboard. 

I created two items, and linked respective channels of these things to it.
 
{{< figure
    src="/images/openHAB-items.png"
    alt="openHAB Items configuration showing linked channels"
    caption="Linking the raw hardware channels to functional openHAB Items"
    class="ma0 w-75"
>}}

Once those Items were configured and worked, I now had to add them to a dashboard layout. Using the built-in layout manager, I dropped both controls onto a dedicated landing page for some quick testing.

{{< figure
    src="/images/openHAB-pages.png"
    alt="Custom control page layout in openHAB UI"
    caption="OpenHAB Landing page with two items and controls"
    class="ma0 w-75"
>}}

The absolute best part of setting it up this way? It transfers perfectly over to the native mobile app ecosystem. Opening up the openHAB mobile application syncs the new page instantly over the local network layer. 

As you can see, the page renders beautifully on mobile, giving me a snappy, real-time interface to trigger one of the light bulbs in my room.

{{< figure
    src="/images/openHAB-mobile.png"
    alt="openHAB mobile application showing the room dashboard"
    caption="The finalized UI layout running natively on the openHAB mobile application"
    class="ma0 w-75"
>}}


---

## Swiss-Army Debugging Snippets

While assembling the custom discovery bridge and monitoring raw JSON parameters passing back and forth across Mosquitto, besides reading through log files, these commands below were incredibly helpful:

### Sniffing Live MQTT Traffic
{{< highlight bash >}}
# Watch targeted setting adjustments
mosquitto_sub -h 127.0.0.1 -v -t "zigbee2mqtt/Spot am 3D Drucker/set"

# Intercept all status broadcasts for a specific device
mosquitto_sub -h 127.0.0.1 -v -t "zigbee2mqtt/Spot am 3D Drucker/+"
{{< /highlight >}}

### Forcing Test Payload Broadcasts
{{< highlight bash >}}
mosquitto_pub -h 127.0.0.1 -t "zigbee2mqtt/Spot am 3D Drucker/set" -m '{"effect": "blink"}'
{{< /highlight >}}

### Managing Stuck Items via the openHAB Karaf Console
{{< highlight bash >}}
# Connect to console environment
ssh openhab@localhost -p 8101

# Inspect active things
openhab:things list

# Wipe a faulty discovery entity manually
openhab:things remove sony:scalar:fcf1523cd795

# Find and uninstall a rogue binding bundle
openhab> bundle:list | grep -i sony
# 369 | Active  |  80 | 4.3.0.202412181559    | openHAB Add-ons :: Bundles :: Sony Binding
openhab> bundle:uninstall 369

# Clear out a corrupted automatic inbox queue
openhab:inbox list
openhab:inbox clear
{{< /highlight >}}

---

## What's Next?

With the system running stably on the fanless hardware, I can easily cruise along with my bridge script until openHAB 5.3 lands with the structural upstream fixes. 

Now that the foundational data lanes are rock solid, what should I tackle next? I'm still figuring out the basic building blocks, so there's a lot left to explore:

* **Creating and linking up items to channels:** Diving deeper into item types (Dimmer, Color, Number) and mastering state transformations so raw payloads display cleanly.
* **Grouping items:** Leveraging openHAB's group item aggregation features to toggle entire rooms at once or compute average temperatures dynamically.
* **Building a meaningful dashboard:** Moving past basic test grids to design intuitive, high-WAF (Web Application Frontend) semantic interfaces for daily mobile and desktop use.
* **Modeling the Semantic Home:** Mapping everything out using openHAB’s Semantic Model (Locations, Equipment, Points) so the system understands exactly *where* a sensor physically sits in the house.
* **Writing foundational rule scripts:** Setting up your first event-driven automations, like utilizing the SONOFF presence sensor to auto-trigger the spot lamps when someone approaches the desk.
* **Persistence & Data Logging:** Setting up rrd4j or InfluxDB storage backends to start tracking temperature and humidity patterns over time without bloating the thin client's flash memory.


