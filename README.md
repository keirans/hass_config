# Home Automation Configuration
This repo contains the configuration and notes around my home automation setup for my Apartment using Home Assistant and a few other technologies.

The original configuration I had before I renovated my apartment can be found in the version 1 release of this repo.
* [Home Automation System - v1.0](https://github.com/keirans/hass_config/releases/tag/1.0)

I did an internal presentation at my work for a bit of fun that you can find the deck for below. This talks about my setup and the wider IOT ecosystem and challenges in the context of automated homes.

  * [The Internet of Things and the Modern Household](docs/IOT_Workshop_Presentation.pdf)

## Equipment and build list

I currently have the following equipment and components

| Component | Function |
|------------|----------|
| <ul><li>Intel NUC Mini PC</li></ul> | <ul><li>Base OS runs Ubuntu Linux with Docker CE</li><li>Home Assistant Core running in Docker</li><li>Model: BOXNUC7CJYSAL4 (4GB + 32GB)</li></ul> |
| <ul><li>USB Zigbee Interface</li></ul> | <ul><li>Provides Home Assistant native Zigbee device support via the ZHA Intergration</li><li>Model : HUBBZB-1</li></ul> |
| <ul><li>Phillips HUE Bridge v2</li></ul> | <ul><li>Provides all HUE functionality</li><li>Phillips HUE Bulbs (5)</li><li>Phillips HUE Light Strips (2)</li><li>HUE Switches (6) </li><li>HUE Movement sensors (2)</li><li>IKEA Tradfi downlights (HUE Compatible) (24)</li></ul> |
| <ul><li>Xiaomi Aquara</li></ul> | <ul><li>All devices are integrated via the Home Assistant Zigbee USB Interface (ZHA)</li><li>Door and Window Sensors (5)</li><li>Temp and Humidity Sensors (5)</li><li>Motion sensors (1)</li><li>Smart Power Plugs (1)</li></ul> |
| <ul><li>Xiaomi Rockrobo Vacuum Cleaner</li></ul> | <ul><li>Automatic cleaning System</li></ul> |
| <ul><li>Xiaomi Flower sensors</li></ul> | <ul><li>Light, Temprature, Water and conductivity sensors for Chilli Plants</li></ul> |
| <ul><li>Xiaomi Dafang cameras</li></ul> | <ul><li>Internal cameras for security and monitoring running custom firmware</li></ul> |
| <ul><li>Daikin Air Conditioner</li></ul> | <ul><li>3Kw Split system with Daikin WiFi Controller</li></ul> |
| <ul><li>Broadcom IR Blaster</li></ul> | <ul><li>Programmable Universal Remote for control of IR Devices such as Fans</li></ul> |
| <ul><li>Google Home Devices</li></ul> | <ul><li>Google Chromecasts</li><li>Google Home voice controlled assistants</li></ul> |
| <ul><li>Synology NAS</li></ul> | <ul><li>Local Storage and Plex Server</li></ul> |
| <ul><li>Unifi Network equipment</li></ul> | <ul><li>USG Firewall</li><li>AC Pro WiFi AP</li><li>Managed Switch</li><li>Cloudkey v1</li></ul> |
| <ul><li>Nabu Casa Subscription</li></ul> | <ul><li>Remote Access, Google Home integration and other functionality</li></ul> |


## Configuration and Integration

### Home Assistant Docker Compose file

I run Home Assistant Core via docker on Ubuntu using Docker Compose

I map through the Bluetooth and USB Zigbee devices into the container so they can be used.

I store the config dir on a volume to make upgrades and backups simple.

```
version: '3'

services:
  homeassistant:
    container_name: home-assistant
    image: homeassistant/home-assistant:stable
    volumes:
      - /opt/hass/config/:/config
    devices:
      - /dev/ttyUSB0:/dev/ttyUSB0
      - /dev/ttyUSB1:/dev/ttyUSB1
      - /dev/ttyACM0:/dev/ttyACM0
    privileged: true
    environment:
      - TZ=Australia/Sydney
    restart: always
    network_mode: host
```

### Pi-Hole Docker Compose file

I run Pi-Hole as a whole network ad blocking solution via docker on Ubuntu using Docker Compose

I also use the Pi-Hole API and Intgration to expose metrics and setup switches to control it via Home Assistant (detailed further on)

```
version: "3"
# More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp"
      - "80:80/tcp"
      - "443:443/tcp"
    environment:
      TZ: 'Australia/Sydney'
      WEBPASSWORD: changeme
    # Volumes store your data between container upgrades
    volumes:
      - '/opt/pihole/etc-pihole/:/etc/pihole/'
      - '/opt/pihole/etc-dnsmasq.d/:/etc/dnsmasq.d/'
    dns:
      - 127.0.0.1
      - 8.8.8.8
    # Recommended but not required (DHCP needs NET_ADMIN)
    #   https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
    #cap_add:
    #  - NET_ADMIN
    restart: unless-stopped
```

### HUE Lighting intergration and approach
I have all HUE and HUE compatible lighting and accessories integrated directly with the HUE Bridge, and then have the Hue Bridge integrated with Home Assistant via the HUE integration. The reason I have done this is so that all the lights function as required if my Home Assistant instance is offline or is being upgraded.

### Route53 Dynamic DNS
I use the Route53 Integration to provide Dynamic DNS records for my home so i can VPN in.

### Android Application for presence detection
I use the [Home Assistant Android Application](https://play.google.com/store/apps/details?id=io.homeassistant.companion.android&hl=en_AU) in conjunction with Nabucasa to provide remote access, presence detection and phone metrics.

### Xiaomi Rockrobo Vacuum Cleaner
I invested in this for the novelty factor, however given my apartment is a single level, it's actually amazing at keeping my place tidy, much more than I expected.

Initally I had the devices rooted using the process below, however now I have it running the custom firmware called [Valetudo - Free your vacuum from the cloud](https://github.com/Hypfer/Valetudo). Once installed, I found that it worked perfectly, gave me more local control options and functionality as well as removing all remote communication to China.

Once Valetudo is installed and configured with Home Assistant, you can also voice control it using the google assistant / cloud integration as it now exposes the vacuum domain. 

I still encourage you to check out the presentation at CCC about this initial reverse engineering work, it's great -  [34C3 - Unleash your smart-home devices: Vacuum Cleaning Robot Hacking](https://www.youtube.com/watch?v=uhyM-bhzFsI),


### Xiaomi Flower sensors
I use the Xiaomi sensors to get some light , tempreature and other metrics for around the chilli plants that I grow both indoors and out, however one thing I found was that using these sensors really wasn't suitable for my gardening requirements, and so I picked up a few of the "Mi Flora" plant sensors.

These are now owned by Xiaomi, and I don't think they are fully integrated with the Mi Home application as yet (and may never be) , and as such they use their own application to configure and use them. To say the Mi Flora application is garbage is an understatement, and gave up trying to reliably use it, opting to integrate it directly with Home Assistant on  using the below documentation. This has been extremely stable and providing much more metrics about the health of the plants including moisture, light and fertility.

* https://www.home-assistant.io/components/sensor.miflora/

It is worth noting that these devices communicate using Bluetooh Low Energy rather than Zigbee, so your Home Assistant platform must have a Bluetooth interface, for those running HASS on the Pi3 this does just work, in my case i pass through the NUCs Bluetooth interface into the Docker container.

### Temperature and Humidity sensors
I have a number of the temprature and humidity sensors throughout my apartment which enables me to understand more about the temprature of the rooms.

Given the automation capabilities of Home Assistant, its quite simple to have automations using these sensors in which notifications or climate control actions trigger based on certain temprature events.

### Xiaomi Cameras
I have a set of 2 Xiaomi Dafang	cameras, that i have running custom firmware.

I haven't used these cameras without the custom firmware as they tie in to the Xiaomi cloud / Mi Home ecosystem and i'm not overly thrilled with the security and privacy implications. It's pretty easy to flash the custom firmware yourself if you are relatively familiar with Linux.

I got in early on these cameras when the Firmware was quite experimental, and it's quite impressive to see how much the custom firmware has evolved, it's support for movement, SSL and MQTT great and seems to mature with each release. I currently use them with the ffmpeg camera support, and will likely move them over to MQTT later on once I have my head around it.

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

### Broadcom IR Blaster
I have a Broadcom Mini 3 IR blaster that I use to turn on the Kambrook Tower Fan I have in my bedroom (KFA837 Arctic LED Display Tower Fan just incase you have this model and want to use my IR codes)

This is possible because I've loaded the IR remote codes for the fan into home assistant, then trigger them as a switch to turn it on.

What is great about this is that the switch domain is exposed to the Home Assistant cloud integration for google home, so I can now control it using google voice actions such as "OK Google - Turn on bedroom fan" and it works great.

* https://www.home-assistant.io/integrations/broadlink/


### Xiaomi Pedestal fan
I found that the IR control of my bedroom fan wasn't as nice as I'd like it, so I have replaced it with 2 x Xiaomi Pedestal fans that I move throughout the house.

* Xiaomi Smartmi DC Conversion Pedestal Fan 2S (zhimi.fan.za4)
* Xiaomi Mijia Frequency Conversion Floor Fan 1X (dmaker.fan.p5)

These fans look great and work well, and can be controlled via [This custom component](https://github.com/syssi/xiaomi_fan)

It's not perfect, however most core functionality is supported, and I'm hoping that we will see this functionality rolled into Home Assistant Core in the not too distant future.


### Google Home and Home Assistant Cloud
I use the Google Home & Assistant ecosystem for media playing, voice control and providing an alternative interface to automation through the google home android app.

I have te following home devices and it all works a treat with  vacuum, switches, light and climate domains

* 2 x Google Home Mini's
* 1 x Google Home
* 1 x Google Chromecast


## Automations, Scripts and Home Assistant Configurations

### Pi-Hole automation

I pull in the metrics from Pi-Hole using the Pi-Hole Integration, and then I setup a command line switch that lets me turn on/off the ad-blocker via it's API from the Home Assistant interface as well as via Google home (Including Voice Control)

```
pi_hole:
  host: localhost:80
  ssl: false
```

```
switch:
  -  platform: command_line
     switches:
       pihole_switch:
         command_on: "/usr/bin/curl -X GET 'http://10.0.2.17/admin/api.php?enable&auth=<TOKEN>'"
         command_off: "/usr/bin/curl -X GET 'http://10.0.2.17/admin/api.php?disable=5400&auth=<TOKEN>'"
         command_state: "/usr/bin/curl -X GET 'http://10.0.2.17/admin/api.php?status&auth=<TOKEN>'"
         value_template: "{{ value_json.status == 'enabled' }}"
```

### Vaccuum Metrics

I pull out the Vaccuum metrics into sensors to build dashboards in the Home Assistant interface.

```
sensor:
  - platform: template
    sensors:

      vacuum_total_cleaned_area:
        value_template: '{{ states.vacuum.xiaomi_vacuum_cleaner.attributes["total_cleaned_area"] }}'
        friendly_name: 'Total Cleaned Area'
        unit_of_measurement: 'MSq'

      vacuum_total_cleaning_time:
        value_template: '{{ states.vacuum.xiaomi_vacuum_cleaner.attributes["total_cleaning_time"] }}'
        friendly_name: 'Total Cleaning Time'
        unit_of_measurement: 'Mins'

      vacuum_cleaning_count:
        value_template: '{{ states.vacuum.xiaomi_vacuum_cleaner.attributes["cleaning_count"] }}'
        friendly_name: 'Total Cleaning Count'
        unit_of_measurement: 'Cleans'

      vacuum_battery_level:
        value_template: '{{ states.vacuum.xiaomi_vacuum_cleaner.attributes["battery_level"] }}'
        friendly_name: 'Battery Level'
        unit_of_measurement: '%'

      vacuum_fan_speed:
        value_template: '{{ states.vacuum.xiaomi_vacuum_cleaner.attributes["fan_speed"] }}'
        friendly_name: 'Fan Speed'
        unit_of_measurement: ''
```

### Turn off AC when Doors are open Automations

I use the following automation to turn off the air conditioning if the balcony door has been open for more than 2 minutes.

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

## Additional Notes

### Integrating generic security systems
A friend of mine had a security camera system in a home he purchased that used analog 1080P cameras back to a generic Korean branded, Chinese made DVR Style box.
I set myself the goal of getting these cameras into home assistant and I'm documenting these here in the event that others may discover this and find it useful.

The only read information about this device was what I could derive in the administration interface as per the below, and extensive searching on the internet didn't lead me to much outside some generic Korean and Chinese pages and a relatively useless user guide.

| What | Value |
|------------|----------|
| DVR Model |  04CH / 060FPS. |
| WEB Version | WEB_v2.7.3_1.0.0.56_140627 |
| Mac Address Lookup |  Itx Security |


The system itself worked well via the web interface, providing real time streams, and the mobile application Nviewer was able to discover all 4 camera streams when pointed at the device.

Looking at the developer mode of the web interface, I was able to find some API endpoints similar to the below, and further googling showed that this software was common across generic camera systems being made in Asia, however I couldnt find extensive documentation about these, nor any security vulnerabilities that would lead me to gaining shell access to the device.


* ```http://username:password@<DEVICE>/cgi-bin/webra_fcgi.fcgi?api=get_live.live.status```
  * Return the status of the device and the current session
* ```http://username:password@<DEVICE>/cgi-bin/webra_fcgi.fcgi?api=get_jpeg_raw&chno=<CHANNEL>```
  * Return a JPG image from channel <CHANNEL> , in this case, there are 4 channels representing 4 cameras 0-3

There was also a RTSP port set in the configuration interface, however using common ONVIF tools to try and discover the camera streams, I was unable to get these discovered.
The next step I made was to TCP dump the communication from the mobile device and the box during setup of the Nviewer application, as I knew that it could discover the streams nicely.

Doing this on my Unifi USG provided me a pcap file I ran through wireshark, however I was unable to derive the streams from this PCAP as the communication between them was encoded in a way I couldnt quite figure out. While I was thinking about this, I did some googling for other tools to handle the discovery of the streams and discovered CameraRadar.

CamaraRadar is a RTSP access tool that can be used to discover and also brute force RTSP streams on a camera or security system endpoint.
* https://github.com/Ullaakut/cameradar
* https://hub.docker.com/r/ullaakut/cameradar

As the tool is available as a docker image, I quickly pulled it down, updated the configuration files to contain my username and password combination and ran it against the endpoint. I was expecting to not get much, however it returned 2 routes that when combined into an RTSP URL and played in VLC stream worked !

```
root@ubuntu:/tmp# docker run -v /tmp:/tmp --net=host -t ullaakut/cameradar --custom-credentials="/tmp/credentials.json" -p 5554 -t 220.233.83.83
Unable to find image 'ullaakut/cameradar:latest' locally
latest: Pulling from ullaakut/cameradar
<SNIP>
Digest: sha256:ea093ec7e8bee51c150cbedffdf52f6636f7c55a595f578c82f06bc9a4c8d8a5
Status: Downloaded newer image for ullaakut/cameradar:latest
Loading credentials...ok
Loading routes...ok
Scanning the network...ok
Attacking routes of 1 streams...

ok
Attempting to detect authentication methods of 1 streams...ok
Attacking credentials of 1 streams...ok
Validating that streams are accessible...ok
Second round of attacks...ok
Validating that streams are accessible...ok
✖       Admin panel URL:        http://<DEVICE>/ You can use this URL to try attacking the camera's admin panel instead.
        Available:              ✖
        IP address:             <DEVICE>
        RTSP port:              5554
        Auth type:              digest
        Username:               <redacted>
        Password:               <redacted>
        RTSP routes:
                              /live/main0
                              /live/main
```

From this, I was able to derive the following RTSP routes for all the cameras

* rtsp://username:password@<DEVICE>:5554/live/main0
* rtsp://username:password@<DEVICE>:5554/live/second0

Where 0 is the ID for each camera 0 through to 3.

We can then pull these cameras into Home Assistant with the following configuration file entries and display them in lovelace.

```
camera:
  - platform: generic
    name: Front
    still_image_url: http://username:password@<DEVICE>/cgi-bin/webra_fcgi.fcgi?api=get_jpeg_raw&chno=0
    stream_source: rtsp://username:password@<DEVICE>:5554/live/second0

  - platform: generic
    name: Side
    still_image_url: http://username:password@<DEVICE>/cgi-bin/webra_fcgi.fcgi?api=get_jpeg_raw&chno=1
    stream_source: rtsp://username:password@<DEVICE>:5554/live/second1

  - platform: generic
    name: Rear
    still_image_url: http://username:password@<DEVICE>/cgi-bin/webra_fcgi.fcgi?api=get_jpeg_raw&chno=2
    stream_source: rtsp://username:password@<DEVICE>:5554/live/second2

  - platform: generic
    name: Garage
    still_image_url: http://username:password@<DEVICE>/cgi-bin/webra_fcgi.fcgi?api=get_jpeg_raw&chno=3
    stream_source: rtsp://username:password@<DEVICE>:5554/live/second3
```

I hope this helps some other people out there trying to figure out this integration.