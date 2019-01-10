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
I don't expose my home assistant platform to the internet, instead, opting to VPN in from my phone, or laptop to hit the web interface or other management platforms on my home network. As such, i need to have a dynamic DNS platform to keep my VPN configuration sane.

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