---
title: 'Creating a simple adblocking DNS server using dnsmasq'
date: '2020-02-01T00:00:00-06:00'
draft: false
tags: ['linux', 'networking']
categories: ['notes']
keywords: ['adblock', 'adblocking dns server', 'dns server']
description: "How to set up a simple adblocking DNS server using dnsmasq and ad-blocking utilities."
authors: Colin
---

This guide will show you how to create a minimalistic adblocking DNS server. This is a great alternative to [Pi-Hole](https://pi-hole.net), which requires being run on a Raspberry Pi or in a Docker container.

<!--more-->

First, install `dnsmasq` using your distribution's package manager.

Next, replace the contents of `/etc/dnsmasq.conf` with the following:

    domain-needed
    bogus-priv
    no-resolv
    server=8.8.8.8
    server=8.8.4.4
    interface=eth0
    listen-address=127.0.0.1
    cache-size=10000
    local-ttl=300
    addn-hosts=/etc/hosts-block

*Note: you can remove the `interface` and `listen-address` lines if you want `dnsmasq` to listen on all interfaces.*

Next, you will need to download an ad server list. We will be using Peter Lowe's [adservers](https://pgl.yoyo.org/adservers/) list in this example, but you can use other ones, such as [this one](http://winhelp2002.mvps.org/hosts.txt) available to use.

    curl "https://pgl.yoyo.org/adservers/serverlist.php?hostformat=hosts&showintro=0" -o /tmp/hosts-block
    sudo mv /tmp/hosts-block /etc

Finally, start `dnsmasq`

    # Alpine Linux
    rc-service dnsmasq start
    
    # systemd distros (Debian, Ubuntu, Fedora, etc)
    systemctl start dnsmasq 

That's it! Enjoy your new ad-blocking DNS server. Remember to set your routers' DHCP server to hand out the IP of your new DNS server.
