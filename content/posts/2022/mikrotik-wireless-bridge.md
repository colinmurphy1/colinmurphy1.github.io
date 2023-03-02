---
title: "Creating a wireless bridge using a MikroTik router"
date: 2022-12-29T12:17:12-06:00
draft: false
description: "How to create a simple wireless bridge with a MikroTik router."
tags:
  - networking
  - mikrotik
aliases:
  - /posts/2022/12/creating-a-wireless-bridge-using-a-mikrotik-router/
---

I have some devices on my local network that are wired-only, or have unreliable
Wi-Fi. Instead of running ethernet to the room that contains these devices,
I opted to create a wireless bridge using a MikroTik hAP AC<sup>2</sup>
wireless router I had lying around. When configured as a wireless bridge, the
MikroTik router will connect a wired network to a wireless network.

Additionally, as this works as a bridge, there is no sort of NAT (network
address translation) running, so it will appear on the same network as the rest
of your devices. You also will not need to configure any static routes,
seamlessly integrating the wired devices connected to the router with the rest
of your network.

I have found that creating a wireless bridge with a MikroTik router works much
better than using a consumer grade router in bridge mode. On an older ASUS
RT-AC1900P (a variant of the popular RT-AC68U sold at Best Buy and other
retailers), I was only getting about 25MB/s throughput. On the MikroTik, I
could easily get near to over 100MB/s two floors away from my router. That is a
huge difference, especially considering the HAP AC<sup>2</sup> does not have
external wireless antennas!

## WinBox console

Before proceeding, I recommend using Mikrotik's GUI tool WinBox to configure
the router. We will be using this tool as it allows configuring the router
without it having any interfaces configured with an IP address. 

To connect to your router, plug it in to the same switch as your computer, or
directly into your computer's network port. Once that is done, open up WinBox
and select the **Neighbors** tab.

![Winbox showing neighbors](/img/WinBox-Neighbors.png)

Once connected using Winbox, we will configure the router using the command
line interface. This can be launched by selecting the **New Terminal** option
in WinBox.

![Winbox terminal](/img/Winbox-terminal.png)

Inside this terminal, you will type in the below commands to configure the
router.

## Factory resetting the router

The default configuration of a hAP ac<sup>2</sup> is set up to work as a SOHO
router, with network address translation and other features enabled. This is
not optimal for our use case, so we will need to factory reset this router to
an empty configuration. You can reset the configuration of the router by
running the below command:

    /system/reset-configuration no-defaults=yes
    Dangerous! Reset anyway? [y/N]: y

This will reset the router to an empty configuration with no interfaces
configured. This will make the configuration in the following steps a lot
easier.

Once the router reboots, connect to it again using WinBox the same way as
before. If you had set an administrator password, it has likely been reset back
to the default.

## Create a bridge interface

We will first need to configure a bridge. The bridge will link all interfaces
on the device, including the two wireless radios on the hAP AC<sup>2</sup>.

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

## Enable DHCP on the bridge

The next step is to enable the DHCP client on the `bridge1` interface. With
DHCP enabled, the router will automatically get an IP address once it is
connected to a wireless network. You'll use this IP address to manage the
router if you ever need to make a configuration change.

    /ip dhcp-client
    add interface=bridge1

With the bridge interface configured, it is now time to set up a wireless
profile so the router can connect to your wireless network.

## Configure wireless profiles

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
configuration as 5GHz offers better bandwidth, especially as we will be
connecting using 802.11ac Wi-Fi. These settings are used for the United States,
so you will probably want to change this to your country's
[regulatory domain][0].

**One thing worth noting**, MikroTik [does not recommend][1] using the
`station-psuedobridge` wireless mode as it does not support layer 2 bridging.
MikroTik only supports L2 bridging if the access point the router is connecting
to is also MikroTik. In my case, I'm not using a MikroTik access point, so it is
fine to use this mode. If any MikroTik gurus know of a better way to do this
and it still uses a bridge and no NAT, I'm all ears! 😉

## Disable unnecessary services

Next, let's disable some unnecessary services to secure the router:

    /ip service
    set telnet disabled=yes
    set ftp disabled=yes
    set api disabled=yes
    set api-ssl disabled=yes

    /tool bandwidth-server
    set enabled=no

This will leave only the HTTP, SSH, and WinBox services enabled improving the
security of the router a bit.

## Finishing touches

Finally, we will configure the timezone. This is not necessary, but is useful if
you will check the logs of the router.

    /system clock
    set time-zone-name=America/Chicago

An optional feature you may find useful is to enable the usr led. This led can
be configured to blink if there is any wireless network activity using the
below command:

    /system leds
    add interface=wlan2 leds=user-led type=wireless-status

## Full configuration

For reference, here is the full configuration of the router:

    /interface bridge
    add name=bridge1
    /interface wireless security-profiles
    set [ find default=yes ] supplicant-identity=MikroTik
    add authentication-types=wpa2-psk mode=dynamic-keys name=profile1 supplicant-identity="" wpa2-pre-shared-key="YourWifiPassword"
    /interface wireless
    set [ find default-name=wlan1 ] band=2ghz-onlyn country="united states3" frequency=auto installation=indoor mode=\
        station-pseudobridge security-profile=profile1 ssid="MySSID"
    set [ find default-name=wlan2 ] band=5ghz-a/n/ac channel-width=20/40/80mhz-Ceee disabled=no frequency=auto mode=\
        station-pseudobridge security-profile=profile1 ssid="MySSID"
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
    set api disabled=yes
    set api-ssl disabled=yes
    /system clock
    set time-zone-name=America/Chicago
    /system leds
    add interface=wlan2 leds=user-led type=wireless-status
    /tool bandwidth-server
    set enabled=no

[0]:https://networktik.com/mikrotik-frequency-modes/
[1]:https://wiki.mikrotik.com/wiki/Manual:Wireless_Station_Modes#Mode_station-pseudobridge
