# Home Automation Configuration
This repo contains the configuration and notes around my home automation setup for my Apartment using Home Assistant and a few other technologies.

## Equipment

I currently have the following equipment configured.

### Raspberry Pi 3
I run [Home Assistant via HASS](https://www.home-assistant.io/) on a Raspberry Pi 3 and it acts as the central hub for a variety of home automation and other IOT components.


### Phillips Hue Lighting
This started the obession, I bought a few of these to tinker with, and it got me interested in all things IOT.

At the moment, I have a mixture of;

* Light bulbs
* Light Strips
* Switches
* Movement sensors

The home assistant integration is very mature, and it does just work.

I use the motion sensors in my living room to turn on ambient lighting when required, primarilly in winter and later at night in summer if/when required.

### Xiaomi Aquara

I have an Xiaomi Aquara kit which has great Home Assistant support and I use a number of the sensors in the house to measure and control various things which I'll go into a bit more detail below.

If you want to understand more about these sensors and actuators there is a great overview of these products on the following Youtube video: [Xiaomi Smart Home Products Explained
](https://www.youtube.com/watch?v=hpb-ZiQvuZ8) and the configuration steps can be found at the  following [Documentation page](https://www.home-assistant.io/components/xiaomi_aqara/)


#### Xiaomi Smart Power Plugs
I use these to control a heated seedling matt for germinating chilli seeds during the Winter.

This plug allows me to remotely turn it on / off in the event that I need to (primarilly due to fire and safety concerns), and as I have a better grasp of the home automation landscape now, I'll use the temprature sensors I have installed to automatically turn on/off the heater when the ambient or soil temprature are below certain thresholds.

Another thing I do like about these plugs is that they provide metrics on the power usage of the attached device.


#### Xiaomi Door / Window Sensors
I have a number of these throughout my house to allow me to see if I have left open doors and windows when I am out, this is particularly useful when a storm is coming.

In addtion to this, I use the following automation to turn off the air conditioning if the balcony door has been open for more than 2 minutes.

```
automation:
  - alias: "Balcony Door Open turn off AC"
    trigger:
      platform: state
      entity_id: binary_sensor.door_window_sensor_158d0001d6855d
      from: 'off'
      to: 'on'
      for:
        minutes: 2
    action:
      service: climate.set_operation_mode
      data:
        entity_id: climate.daikinap62514
        operation_mode: "off"
```


#### Xiaomi Movement and Light Sensors
I currently don't use these for much, I stuck them near some of my Chilli Plants to measure the light levels before I had the plant sensors and found they weren't really suited for this (unsurprisingly), and as my Hue setup has motion sensors that work well for light control, I'll likely put them to other use in the future.

#### Xiaomi IOT Button
I used this to get the hang of Home Assistant automation capabilities, turning on Hue lights with Xiaomi Buttons is a good use case. I think I'll be using this to enable "Movie Mode" in my living room, in which movement sensors are disabled for say 3 hours and the lights are dimmed and set to a preset configuration.


### Xiaomi Rockrobo Vacuum Cleaner
I invested in this for the novelty factor, however given my apartment is a single level, it's actually amazing at keeping my place tidy, much more than I expected.

There are a few ways to get it integrated with Home Assistant, however the most robust way that I have found is to root it and extract the API key.

To root it, check out the presentation these guys did at CCC -  [34C3 - Unleash your smart-home devices: Vacuum Cleaning Robot Hacking](https://www.youtube.com/watch?v=uhyM-bhzFsI),and then I used their docs to root my vaccuum, and run the following commands to extract the API token to integrate it into Home Assistant.

```
root@rockrobo:~# printf $(cat /mnt/data/miio/device.token) | xxd -p
```

You can find more about this command on this github issue: https://github.com/dgiese/dustcloud/issues/12 and there is a good review of this model on youtube here: [Xiaomi Mi Robot Vacuum Review - Raising The Bar at an Affordable Price
](https://www.youtube.com/watch?v=eWLWO5AxAHo)

Once this is configured, you can also voice control it using the google assistant / cloud integration as it now exposes the vacuum domain. 

### Xiaomi Flower sensors
As I mentioned above, I use the Xiaomi sensors to get some light , tempreature and other metrics for around the chilli plants that I grow both indoors and out, however one thing I found was that using these sensors really wasn't suitable for my gardening requirements, and so I picked up a few of the "Mi Flora" plant sensors.

These are now owned by Xiaomi, and I don't think they are fully integrated with the Mi Home application as yet (and may never be) , and as such they use their own application to configure and use them. To say the Mi Flora application is garbage is an understatement, and gave up trying to reliably use it, opting to integrate it directly with Home Assistant on the Pi3 using the below documentation. This has been extremely stable and providing much more metrics about the health of the plants including moisture, light and fertility.

* https://www.home-assistant.io/components/sensor.miflora/

It is worth noting that these devices communicate using Bluetooh Low Energy rather than Zigbee, so your Home Assistant platform must have a Bluetooth interface, for those running HASS on the Pi3 this does just work.

### Temperature and Humidity sensors
I have a number of the temprature and humidity sensors throughout my apartment which enables me to understand more about the temprature of the rooms.

Given the automation capabilities of Home Assistant, its quite simple to have automations using these sensors in which notifications or climate control actions trigger based on certain temprature events.

### Xiaomi Cameras
I have a set of 2 Xiaomi Dafang	cameras, that i have running custom firmware.

I haven't used these cameras without the custom firmware as they tie in to the Xiaomi cloud / Mi Home ecosystem and i'm not overly thrilled with the security and privacy implications. It's pretty easy to flash the custom firmware yourself if you are relatively familiar with Linux.

I got in early on these cameras when the Firmware was quite experimental, and it's quite impressive to see how much the custom firmware has evolved, it's support for movement, SSL and MQTT is really quite impressive. I currently use them with the ffmpeg camera support, and will likely move them over to MQTT later on once I have my head around it.

More information about these cameras and the custom firmware can be found at the links below;

* [Hacking a $25 IOT Camera to do more than it's worth](https://hackernoon.com/hacking-a-25-iot-camera-to-do-more-than-its-worth-41a8d4dc805c)
* [Xiaomi Dafang Hacks - Github.com](https://github.com/EliasKotlyar/Xiaomi-Dafang-Hacks)

### Daikin Air Conditioner
Unbeknownst to me, this was actually my very first IOT purchase. When adding this Daikin AC unit to my home to help me survive the brutal Sydney summer heat, I opted to get the wifi controller to be able to control it from an application both locally and remotely. 

I expected this to be pretty locked down, and unlikely to be able to be integrated into a wider automation platform and did some poking around online and found that the local wifi communication between android and iphone applications is unauthenitcated and in plaintext which in the context of IOT is generally bad, however the benefit of this is that the API has already been extensively reverse engineered and a Python interface for it built which has been dropped into Home Assistant, making it a first class citizen that "just works" via auto discovery.

To make things even better, recent changes and enhancements with this component have made it work extremely well with the Google Home climate control capability in the home application as well as voice control (I'd suggest signing up for the home assistant cloud service to make things a little easier to get working).

There is a number of related projects that I used as references when trying to understand how all this went together, and you can check them out below if you have this equipment.

* https://github.com/ael-code/daikin-control
* https://www.home-assistant.io/components/daikin/
* https://pypi.org/project/pydaikin/
* https://github.com/Apollon77/daikin-controller


### IR Blaster

broadcom mini 3

https://www.home-assistant.io/components/switch.broadlink/

Garbage application, does however work great with Home assistant.



### Kambrook Tower Fan + IR Remote

I have a Kambrook KFA837 Arctic LED Display Tower Fan Black in the bedroom, as it has an IR controller, i've loaded it's buttons into the IR blaster as switches, and these can be controlled via the home assistant web interface as well as google home.

### Google Home and Home Assistant Cloud

* 2 x Google Home Mini's
* 1 x Google Home
* 1 x Google Chromecast

Voice control via Home Assistant Cloud works a treat with vacuum, switches, light domains

Mainly use this as group speakers for podcastss and music

### Samsung TV

### Plex

## Software components

### Unifi Home Network

* [Here's What This Ubiquiti UniFi Stuff Is All About](https://www.troyhunt.com/heres-what-this-ubiquiti-unifi-stuff-is-all-about/)
* [Wiring a home network from the ground-up with Ubiquiti](https://www.troyhunt.com/wiring-a-home-network-from-the-ground-up-with-ubiquiti/)
* [Ubiquiti all the things: how I finally fixed my dodgy wifi](https://www.troyhunt.com/ubiquiti-all-the-things-how-i-finally-fixed-my-dodgy-wifi/)

### DNS Filtering and Blocking with Pi-Hole
I use the amazing Pi-Hole as a network wide ad blocker, as well as a way to provide DNS blocking and visibility capabilities for my network, specifically the IOT devices.

Installed as a Hass add-on

* [Troy Hunt: Mmm... Pi-hole...](https://www.troyhunt.com/mmm-pi-hole/)
* [Catching and dealing with naughty devices on my home network](https://scotthelme.co.uk/catching-naughty-devices-on-my-home-network/)
* [What really happens on your network]https://pi-hole.net/2018/09/19/what-really-happens-on-your-network-part-eight/


### Route53 for Dynamic DNS
I don't expose my home automation platform to the internet, instead, opting to VPN in from my phone, or laptop to hit the web interface or other management platforms on my home network. As such, i need to have a dynamic DNS platform to keep my VPN configuration sane.

I initally wrote home assistant add-on to handle all the Route 53 DNS configuration for my home up to date, then when i had some time, i added the native support via a PR into Home Assistant natively, which everyone can now use out of the box.

You can check them out below:

* https://www.home-assistant.io/components/route53/
* https://github.com/keirans/hassio_route53 (Deprecated)


### Presence Detection with Unifi

I have presence detction setup using: 

* https://www.home-assistant.io/components/device_tracker.unifi/


### SSH Add-On

I have the SSH Add-on installed for management.


### MQTT Add-On

I have the MQTT Add-on installed for management.


### Synology NAS sensors


## Network Topology, Network Segregation & Wifi Configuration



### mDNS Configuration
I have mdns enabled on the USG to allow Multicast DNS to function across vlans, this helps for some cross VLAN functionality for some IOT components, it also allows me to use multicast DNS entries published from things like hass in the IOT network, on my internal network which is nice.

### Unifi controller software
https://www.home-assistant.io/blog/2018/04/12/ubiquiti-and-home-assistant/

### Bluetooth LE android app

#### Google Home Application

Use it for aircon control primarilly when on the go.

#### Wifi Pineapple







