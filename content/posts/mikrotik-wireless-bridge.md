---
title: "Creating a wireless bridge using a MikroTik router"
date: 2022-12-29T12:17:12-06:00
draft: false
description: "How to create a simple wireless bridge with a MikroTik router."
tags:
  - networking
  - mikrotik
---

I have some devices on my local network that are wired-only, or have unreliable
Wi-Fi. Instead of running ethernet to the room that contains these devices,
I opted to create a wireless bridge using a MikroTik hAP AC^2 wireless router
I had lying around. For those who are unaware, a wireless bridge is a
router configuration that "bridges", or links, the network connection of a
wired device to a wireless network.

I have found that creating a wireless bridge with a MikroTik router works much
better than using a consumer grade router in bridge mode. On an older ASUS
RT-AC1900P (a variant of the popular RT-AC68U sold at Best Buy and other
retailers), I was only getting about 25MB/s throughput. On the MikroTik, I
could easily get near to over 100MB/s two floors away from my router. That is a
huge difference, especially considering the HAP AC^2 does not have external
wireless antennas!

On a MikroTik router running RouterOS, this configuration is relatively
straightforward. I strongly recommend resetting the router to a default
configuration by running the below command:

    [admin@MikroTik] > /system reset-configuration no-defaults
    Dangerous! Reset anyway? [y/N]: y

This will reset the router to a default configuration that does not contain
any sort of settings.

Once the router reboots, I recommend using WinBox to access the router, as it
allows you to configure it without needing an IP address configured on the
router. Your router will appear in the "Neighbors" tab of the WinBox interface:

![Winbox showing neighbors](/img/WinBox-Neighbors.png)

Once connected using Winbox, let's configure the router using the command line.
This can be launched by selecting the **New Terminal** option in WinBox.

![Winbox terminal](/img/Winbox-terminal.png)

Inside this terminal, you will type in the below commands to configure the
router as a wireless bridge.

We will first need to configure a bridge. The bridge will link all interfaces
on the device, including the two wireless radios on the hAP AC^2.

    /interface bridge
    add name=bridge1

Next, let's add the interfaces:

    /interface bridge port
    add bridge=bridge1 interface=ether1
    add bridge=bridge1 interface=ether2
    add bridge=bridge1 interface=ether3
    add bridge=bridge1 interface=ether4
    add bridge=bridge1 interface=ether5
    add bridge=bridge1 interface=wlan2
    add bridge=bridge1 interface=wlan1

The next step is to enable the DHCP client on the `bridge1` interface. With
DHCP enabled, the router will automatically get an IP address once it is
connected to a wireless network. You'll use this IP address to manage the
router if you ever need to make a configuration change.

    /ip dhcp-client
    add interface=bridge1

With the bridge interface configured, it is now time to set up a wireless
profile so the router can connect to your wireless network.

Create a wireless profile to use WPA2 encryption:

    /interface wireless security-profiles
    add authentication-types=wpa2-psk mode=dynamic-keys name=profile1 supplicant-identity="" wpa2-pre-shared-key="YourWifiPassword"

You will need to add in your WPA2 pre shared key (password) above. 

Next, we will configure the wireless interfaces to use your the security
profile we created, and your SSID.

    /interface wireless
    set [ find default-name=wlan1 ] band=2ghz-onlyn country="united states3" frequency=auto installation=indoor mode=station-pseudobridge security-profile=profile1 ssid="MySSID"
    set [ find default-name=wlan2 ] band=5ghz-a/n/ac channel-width=20/40/80mhz-Ceee disabled=no frequency=auto mode=station-pseudobridge security-profile=profile1 ssid="MySSID"

I will be connecting over 5GHz, so I will be using wlan2, the 5GHz radio. It has
been enabled, as `disabled=no` is set. The 2.4GHz radio is disabled in this 
configuration as 5GHz has better throughput, especially as we will be connecting
using 802.11ac Wi-Fi. These settings are used for the United States, you will
need to figure out your configuration if you live in another country, as yours
may have different regulatory requirements.

**One thing worth noting**, MikroTik [does not recommend][1] using the
`station-psuedobridge` wireless mode as it does not support layer 2 bridging.
MikroTik only supports L2 bridging if the access point the router is connecting
to is also MikroTik. In my case, I'm not using a MikroTik access point, so it is
fine to use this mode. If any MikroTik gurus know of a better way to do this
and it still uses a bridge and no NAT, I'm all ears! ;) 

Next, we will configure the timezone. This is not necessary, but is useful if
you will check the logs of the router.

    /system clock
    set time-zone-name=America/Chicago

A cool feature I like to enable is to blink the usr led on the back of the 
router when there is wireless activity:

    /system leds
    add interface=wlan2 leds=user-led type=wireless-status

Finally, let's disable some unnecessary services to secure the router:

    /ip service
    set telnet disabled=yes
    set ftp disabled=yes

    /tool bandwidth-server
    set enabled=no

There may be some more services you can disable. For example, you could disable
the API and WinBox access if you'd like. I generally like to leave HTTP and SSH
access enabled. 

For reference, here is the full configuration of the router:

    /interface bridge
    add name=bridge1
    /interface wireless security-profiles
    set [ find default=yes ] supplicant-identity=MikroTik
    add authentication-types=wpa2-psk mode=dynamic-keys name=profile1 supplicant-identity="" wpa2-pre-shared-key="YourWifiPassword"
    /interface wireless
    set [ find default-name=wlan1 ] band=2ghz-onlyn country="united states3" frequency=auto installation=indoor mode=station-pseudobridge security-profile=profile1 ssid="MySSID"
    set [ find default-name=wlan2 ] band=5ghz-a/n/ac channel-width=20/40/80mhz-Ceee disabled=no frequency=auto mode=station-pseudobridge security-profile=profile1 ssid="MySSID"
    /interface bridge port
    add bridge=bridge1 interface=ether1
    add bridge=bridge1 interface=ether2
    add bridge=bridge1 interface=ether3
    add bridge=bridge1 interface=ether4
    add bridge=bridge1 interface=ether5
    add bridge=bridge1 interface=wlan2
    add bridge=bridge1 interface=wlan1
    /ip dhcp-client
    add interface=bridge1
    /ip service
    set telnet disabled=yes
    set ftp disabled=yes
    /system clock
    set time-zone-name=America/Chicago
    /system leds
    add interface=wlan2 leds=user-led type=wireless-status
    /tool bandwidth-server
    set enabled=no

[1]:https://wiki.mikrotik.com/wiki/Manual:Wireless_Station_Modes#Mode_station-pseudobridge
