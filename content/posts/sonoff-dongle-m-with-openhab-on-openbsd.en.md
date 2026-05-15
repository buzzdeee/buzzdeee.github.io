---
title: "Zigbee Home Automation on OpenBSD: Sonoff Dongle-M and openHAB"
subtitle: "Zigbee over Ethernet, PoE setup, and the winning Z2M + openHAB stack on OpenBSD"
date: 2026-05-14T19:49:28+02:00
lastmod: 2026-05-14T19:49:28+02:00
draft: false
author: ""
authorLink: ""
description: "Setting up the Sonoff Dongle Max on OpenBSD with openHAB and Zigbee2MQTT, including PoE tips and troubleshooting common connection hurdles."
license: "MIT"
images: [ "/images/sonoff_openbsd.png", "/images/sonoff-dongle-max.jpeg", "/images/sonoff_dongle_zigbee2mqtt_connection_information.png", "/images/Zigbee2mqtt_frontend.png", "/images/wireshark_zigbee_decrypted.png" ]

tags: ["OpenBSD", "Zigbee", "openHAB", "Zigbee2MQTT", "Sonoff"]
categories: ["Smart Home"]

featuredImage: "/images/sonoff_openbsd.png"
featuredImagePreview: "/images/sonoff_openbsd.png"

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
  images: [ "/images/sonoff_openbsd.png", "/images/sonoff-dongle-max.jpeg", "/images/sonoff_dongle_zigbee2mqtt_connection_information.png", "/images/Zigbee2mqtt_frontend.png", "/images/wireshark_zigbee_decrypted.png" ]
  # ...
---

In my [previous post](/en/zigbee-sniffing-on-openbsd/), we looked at sniffing Zigbee traffic using the older [TI CC2531](https://www.ebay.de/itm/404481824304?var=674447321503&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=TICC2531&toolid=10001&mkevt=1). While great for learning, it's not the hardware you want running your actual home. The CC2531 is excellent for sniffing because it simply runs the sniffer firmware—that is what I used it for, and that is what I will keep using it for.

Following my own advice, I bought myself a **[Sonoff Dongle Max](https://sonoff.tech/en-de/products/sonoff-dongle-max-zigbee-thread-poe-dongle-dongle-m)** on [eBay](https://www.ebay.de/itm/388971238285?amdata=enc%3AAQALAAAAoGfYFPkwiKCW4ZNSs2u11xBWFdbfFYp4er2kjyno9qmO593oK669JSSjjTgWvi6tTRNzvYtGUbb3EfOMwwSyITjSDhxkDoFtcCFaXt%2BLbbqQavhhYkKezX99g2LMSYO2N5S0ljwoKg1gnLanibPKbepX68fje0f75JFx9r0FqrNHOksW%2BWM%2BVAIttAcpDD38IEYQS3XIc7r0yK3BY7TAV6w%3D&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Sonoff&toolid=10001&mkevt=1) to act as a proper **Zigbee Coordinator**. In that previous post, I could only see a few Zigbee and other 802.15.4 devices from neighbors, but where is the fun in that? It got me curious and looking to get my own home automation started!

{{< admonition type="tip" title="eBay" >}}
On [eBay](https://www.ebay.de/itm/388971238285?amdata=enc%3AAQALAAAAoGfYFPkwiKCW4ZNSs2u11xBWFdbfFYp4er2kjyno9qmO593oK669JSSjjTgWvi6tTRNzvYtGUbb3EfOMwwSyITjSDhxkDoFtcCFaXt%2BLbbqQavhhYkKezX99g2LMSYO2N5S0ljwoKg1gnLanibPKbepX68fje0f75JFx9r0FqrNHOksW%2BWM%2BVAIttAcpDD38IEYQS3XIc7r0yK3BY7TAV6w%3D&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=Sonoff&toolid=10001&mkevt=1) you have the same price as in their shop, but you can make use of the "Preisvorschlag senden" to get it a few Euros cheaper. Worked for me ;)
{{< /admonition >}}

## Hardware: The Sonoff Dongle Max

{{< figure
    src="/images/sonoff-dongle-max.jpeg"
    alt="Sonoff Dongle Max"
    caption="Sonoff Dongle Max"
    class="ma0 w-75"
>}}

The Sonoff Dongle Max is a versatile beast. Unlike the cheap USB sticks, this version supports:
* **USB Connectivity**
* **Ethernet with PoE (Power over Ethernet)**
* **WiFi**

Their getting started documentation you can find [here](https://dongle.sonoff.tech/guide/dongle-m/donglem-getting-started/).

Connected to USB, the device shows up as a serial port.
{{< highlight bash >}}
uslcom0 at uhub5 port 2 configuration 1 interface 0 "SONOFF SONOFF Dongle Max MG24" rev 2.00/1.00 addr 5
ucom0 at uslcom0 portno 0: usb1.1.00002.0
{{< /highlight >}}

{{< admonition type="tip" title="Why PoE?" >}}
While USB works for testing, it forces your coordinator to live next to your server (likely in a basement or a rack). Using **Zigbee over Ethernet** allows you to place the dongle centrally in your home via a single PoE cable for both power and data. If you, unlike myself, don't have a switch that supports PoE natively, you can get a cheap [PoE Injector](https://www.ebay.de/sch/i.html?_nkw=PoE+Injector&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=PoeInjector&toolid=10001&mkevt=1) on eBay as well.
{{< /admonition >}}


### The "Smart IP" Headache
A quick warning for the factory defaults: The dongle uses "Smart IP." It remembers the very first IP it receives from DHCP and sticks to it like glue, even if your DHCP server tries to assign a different static lease later. 

If you have trouble reaching the device after setting a static lease, you'll hopefully have remembered it's initial IP and connect there, or you need to connect to its internal WiFi AP to manually switch the settings from "Smart IP" to standard DHCP.

This caused me quite a bit of headache ;)

In any case, once you figured that out, you can direct your browser to http://<hostname or IP address> to get to the Sonoff Dongle Max web frontend.

### Finding Connection Information
To configure openHAB or Zigbee2MQTT, you need the specific port strings from the dongle's web interface. In the menu on the left, navigate to **Z2M&ZHA -> Zigbee2MQTT**. From there, you can expand the **USB Connection** and **TCP Connection** sections to get the exact paths and port numbers required for your config files.

{{< figure
    src="/images/sonoff_dongle_zigbee2mqtt_connection_information.png"
    alt="Zigbee2MQTT connection information in the Sonoff Web UI"
    caption="Zigbee2MQTT connection information in the Sonoff Web UI"
    class="ma0 w-75"
>}}

{{< admonition type="tip" title="Connection Info" >}}
For a serial connection via USB, on OpenBSD, the serial device to use is typically `/dev/cuaU0` or alike. With regard to the TCP connection, ensure to use the proper hostname, or stay with the IP address.
{{< /admonition >}}

## Setting up openHAB on OpenBSD

Installing openHAB on OpenBSD is straightforward thanks to the available packages. Once the packages are in place, you're left with starting the service.

{{< highlight bash >}}
pkg_add -i openhab%5 openhab-addons%5
rcctl start openhab
{{< /highlight >}}

After the service starts, point your browser to `http://localhost:8080` (or your server's IP). You'll be greeted by the setup wizard where you'll create your initial **Administrator** account. Once logged in, the first order of business is to head over to **Settings > Add-ons** and install the **Zigbee Binding**.

{{< admonition type="info" title="New to openHAB?" >}}
If you are as new to home automation and openHAB as I am, I highly recommend reading through the official [openHAB Documentation](https://www.openhab.org/docs/). It does a fantastic job of explaining the core concepts like Things, Items, and Channels, which can be a bit overwhelming at first! Therefore take a look at the [Concepts](https://www.openhab.org/docs/concepts/) first, as well as the [Getting Started Tutorial](https://www.openhab.org/docs/tutorial/).
{{< /admonition >}}

## Initial Attempt: The Direct Zigbee Binding Route

Once logged in, my first instinct was to keep it simple and use the native binding:

1. Go to **Settings > Add-ons** and install the **Zigbee Binding**.
2. Navigate to **Settings > Things**, click the blue **+** button, and select the **Zigbee Binding**.
3. Choose **Ember Coordinator** from the list (the correct driver for the Sonoff Dongle-M).
4. In the configuration, I set the port to: `tcp://10.0.0.210:6638`.

I also configured the Baud Rate to `115200` and Flow Control to `None`. However, even after clicking save, the Thing remained stubborn and wouldn't come online. 

To see what was happening under the hood, I bumped up the logs in the Karaf console:

{{< highlight bash >}}
# Connect to Karaf (default pass: habopen)
ssh -p 8101 openhab@localhost
log:set DEBUG org.openhab.binding.zigbee
log:set DEBUG com.zsmartsystems.zigbee
{{< /highlight >}}

The log revealed the wall I had hit:

{{< highlight text >}}
[DEBUG] [nding.zigbee.serial.ZigBeeSerialPort] - Connecting to serial port [tcp://10.0.0.210:6638] at 115200 baud...
[DEBUG] [nding.zigbee.serial.ZigBeeSerialPort] - No communication ports found, cannot connect to [tcp://10.0.0.210:6638]
[ERROR] [zigbee.dongle.ember.ZigBeeDongleEzsp] - EZSP Dongle: Unable to open serial port
{{< /highlight >}}

Looking at the OpenBSD port `README`, there is a significant hint regarding Java serial access:
> "The example of passing `-Dgnu.io.rxtx.SerialPorts` uses the rxtx library, which is currently unsupported on OpenBSD. However, 100% Java implementations such as PureJavaComm and jSerialComm are known to work."

The `README` refers to physical serial ports, not TCP. However, because the TCP method failed immediately and I saw the note about `rxtx` support being missing, I didn't even bother attempting a local serial connection. Time for a more robust architectural approach.


## The Winning Stack: Zigbee2MQTT + Mosquitto

Instead of a direct connection, I moved to a decoupled stack:
**Sonoff (TCP) <-> Zigbee2MQTT <-> Mosquitto (MQTT) <-> openHAB.**

### 1. Install the tools
{{< highlight bash >}}
pkg_add -i mosquitto zigbee2mqtt
{{< /highlight >}}

### 2. Configure Zigbee2MQTT
Edit your `/etc/zigbee2mqtt/configuration.yaml`. Note the `adapter: ember` setting, which is required for the MG24 chip in the Dongle-M.

{{< highlight yaml >}}
version: 5
homeassistant:
  enabled: true
frontend:
  enabled: true
  host: 127.0.0.1
  port: 8081
mqtt:
  base_topic: zigbee2mqtt
  server: mqtt://localhost
serial:
  port: tcp://10.0.0.210:6638
  baudrate: 115200
  adapter: ember
  disable_led: false
  rtscts: false
advanced:
  channel: 25
  log_level: debug
  log_directory: /var/log/zigbee2mqtt
  log_file: zigbee2mqtt_%TIMESTAMP%.log
  log_rotation: true
  log_output:
    - file
  network_key: GENERATE
  pan_id: GENERATE
  ext_pan_id: GENERATE
availability:
  enabled: true
{{< /highlight >}}

If you don't have PoE available, or simply prefer to connect the dongle directly via USB, change your `serial:` configuration to the following:

{{< highlight yaml >}}
serial:
  port: /dev/cuaU0
  adapter: ember
  baudrate: 115200
  disable_led: false
  rtscts: false
{{< /highlight >}}

{{< admonition type="info" title="USB Permissions on OpenBSD" >}}
If you use the USB connection, the `_z2m` user must have permission to access the serial radio device (typically `/dev/cuaU0`). On OpenBSD, you do this by adding the user to the **dialer** group:
{{< /admonition >}}

{{< highlight bash >}}
usermod -G dialer _z2m
{{< /highlight >}}

{{< admonition type="info" title="Zigbee2MQTT Documentation" >}}
Similarly to the openHAB documentation, the [Zigbee2MQTT Documentation](https://www.zigbee2mqtt.io/guide/getting-started/) is definitely worth a read.
{{< /admonition >}}

### Configure Mosquitto
For a basic local installation, the default configuration is sufficient, and no immediate changes are required to get started.

{{< admonition type="info" title="Exploring Mosquitto" >}}
While the defaults work out of the box for this setup, the official [Mosquitto documentation](https://mosquitto.org/documentation/) is an excellent resource if you eventually need to dive into security, bridging, or more complex message routing.
{{< /admonition >}}

### 3. Start the services
{{< highlight bash >}}
rcctl start mosquitto
rcctl start zigbee2mqtt
{{< /highlight >}}

### The Zigbee2MQTT Frontend
Since we enabled the `frontend` in the configuration file, you can manage your network directly through a web browser. By default, it is reachable at `http://127.0.0.1:8081`. This interface is incredibly helpful for pairing new devices, checking the network map, and monitoring the status of your sensors.

{{< figure
    src="/images/Zigbee2mqtt_frontend.png"
    alt="The Zigbee2MQTT Web Frontend"
    caption="The Zigbee2MQTT Web Frontend"
    class="ma0 w-75"
>}}

## Integrating with openHAB

Once Zigbee2MQTT is running, integration is seamless:
1. Install the **MQTT Binding** in openHAB.
2. Add a new **MQTT Broker** Thing, pointing to `127.0.0.1`.
3. Because we enabled `homeassistant: true` in Z2M, your devices are supposed to appearing in the openHAB Inbox automatically as they are paired!

## Bonus: Sniffing the new network
Even though I'm now using this for production, the "sniffer" in me still wants to see the packets. To decrypt the traffic of your new network in Wireshark, you need the Network Key.

At first start, Zigbee2MQTT will update the `GENERATE` string in your `/etc/zigbee2mqtt/configuration.yaml` and it will look like this:

{{< highlight yaml >}}
  network_key:
    - 104
    - 73
    - 101
    - 15
    - 20
    - 56
    - 102
    - 28
    - 132
    - 77
    - 211
    - 116
    - 109
    - 134
    - 9
    - 20
{{< /highlight >}}

{{< admonition type="danger" title="Protect Your Network Key!" >}}
Treat your **Network Key** like your most sensitive password. If someone obtains this key and is within physical range of your home, they can:
* **Decrypt all traffic:** Listen to every sensor event, switch state, and data packet sent across your network.
* **Inject commands:** Remotely control your lights, locks, or any other Zigbee devices without your permission.
* **Sniff passively:** Since Zigbee is a mesh protocol, they don't even need to be "connected" to your Wi-Fi to exploit this; a simple USB sniffer is all it takes.

Only use this key for your own private troubleshooting in Wireshark and **never** include it in a public blog post or support forum!
{{< /admonition >}}

{{< highlight bash >}}
# Your key from configuration.yaml
# - 104 - 73 - 101 - 15 - 20 ...

printf '%02x' 104 73 101 15 20 56 102 28 132 77 211 116 109 134 9 20 && echo
# Output: 6849650f1438661c844dd3746d860914
{{< /highlight >}}

Then using my [TI CC 2531](https://www.ebay.de/itm/404481824304?var=674447321503&mkcid=1&mkrid=707-53477-19255-0&siteid=77&campid=5339147890&customid=TICC2531&toolid=10001&mkevt=1) I can use `whsniff` which made it just Today into the OpenBSD ports tree, to feed IEEE802.15.4 traffic into Wireshark:
{{< highlight bash >}}
sudo pkg_add -i whsniff wireshark
sudo whsniff -c 25 | wireshark -k -i -
{{< /highlight >}}

Once Wireshark is up and running, take that output string from above command and add it to **Wireshark -> Preferences -> Protocols -> Zigbee -> Keys**. Now you can watch your home automation commands fly through the air in plain text!

{{< figure
    src="/images/wireshark_zigbee_decrypted.png"
    alt="Sniffing decrypted Zigbee traffic in Wireshark"
    caption="Sniffing decrypted Zigbee traffic in Wireshark"
    class="ma0 w-75"
>}}

{{< admonition type="info" title="Decoding the Addresses" >}}
When looking at the Wireshark logs, you will notice many packets coming from or going to address **`0x0000`**. In the Zigbee world, this is the standardized short address for the **Network Coordinator** (in my case, my Sonoff Dongle Max).
{{< /admonition >}}

With that, I can watch my home automation commands fly through the air - this time on my own terms!

## Conclusion

Getting this stack running on OpenBSD was a bit of a challenge, but a rewarding one. As I wrap up this initial setup, I'm reminded of a section in the [openHAB Introduction](https://www.openhab.org/docs/#what-you-need-to-know-before-you-start) that rings especially true:

> **What You Need to Know Before You Start**
> 
> When home automation just seems to work, it is always the result of hard work. Home automation is fascinating and requires a considerable investment of your time. Here are some key considerations especially for new users. To be successful, you will need to:
>
> * Start slowly and proceed one step at a time
> * Be prepared to learn
> * Remain flexible in how you want to achieve your goal
> * Celebrate all the small successes

That is exactly what I am doing here, and likely what I will be doing in a number of follow-up posts. Getting the coordinator talking to the stack on OpenBSD is a significant first step and a success for me—one I am definitely going to celebrate.

The documentation also leaves us with one final warning:

> "Lastly, be prepared to start a new hobby: home automation."

This is clearly true, though luckily for me, it has quite a bit of overlap with my other hobbies, making it a very natural extension of them!

Stay tuned—the sensors are on their way.

