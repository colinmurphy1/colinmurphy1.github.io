---
title: "Access your local network from anywhere with WireGuard VPN"
date: 2022-12-29T11:36:52-06:00
draft: false
description: "A guide on how to build a WireGuard VPN that allows you to access local network resources from anywhere"
tags:
  - linux
  - wireguard
  - networking
---

For a very long time, I have been using SSH tunneling to access my selfhosted
services. This works great, but exposes your server to a multitude of threats.
I had to implement additional defenses, such as the Fail2Ban daemon to
automatically ban IPs after multiple failed login attempts.

I decided that a better option would be to simply implement a VPN. I ended up
choosing WireGuard as it's built into the kernel, and offers great throughput
if I needed to transfer files over the VPN.

Here is how I set up a VPN using WireGuard:

## Network topology

![Wireguard example topology](/img/wireguard_example.svg)

Peer1 connects to SERVER over the internet. SERVER has a route to the LAN
(192.168.1.0/24) network, allowing Peer1 to access resources on the LAN.

## Installing WireGuard

The first step to setting up a WireGuard is ensuring you have WireGuard
installed and available to use. On Ubuntu, this can be done by running the
following command:

{{< highlight bash >}}
sudo apt install wireguard
{{</ highlight >}}


Now that WireGuard is installed, we will need to create a key for both peers:
the client and server.

## Creating keys

The following commands will generate a public and private key for each peer.

{{< highlight bash >}}
wg genkey | (umask 0077 && tee server.key) | wg pubkey > server.pub
wg genkey | (umask 0077 && tee peer1.key) | wg pubkey > peer1.pub
{{</ highlight >}}

You should now see a .key and .pub for both server and peer1. The .key will be
the private key, and .pub the public key. 

## Enabling IPv4 forwarding

By default, IPv4 forwarding is disabled on most Linux distributions. You will
need to enable this feature, so iptables can forward packets on to the LAN
interface.

To do this, run these commands:

{{< highlight bash >}}
sysctl net.ipv4.ip_forward=1
{{</ highlight >}}

IPv4 forwarding is now enabled, but this is not a persistent configuration
change. To make this change persistent, you will need to edit the sysctl
configuration files. This is located at `/etc/sysctl.d/99-sysctl.conf`.

Open this file, and uncomment the line that contains this text:
`net.ipv4.ip_forward=1`. Save the file. 

The configuration change will now apply on each boot.

## Creating the server configuration file

Create the configuration folder and file:

{{< highlight bash >}}
mkdir /etc/wireguard
vim /etc/wireguard/wg0.conf
{{</ highlight >}}

Inside of `wg0.conf`, add the following lines:

{{< highlight ini >}}
[Interface]
Address = 192.168.99.1/24
ListenPort = 51820
PrivateKey = SERVER Private Key
PostUp = iptables -A FORWARD -i br0 -o %i -j ACCEPT; iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o br0 -j MASQUERADE
PostDown = iptables -D FORWARD -i br0 -o %i -j ACCEPT; iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o br0 -j MASQUERADE
 
[Peer]
PublicKey = Peer1 public key
AllowedIPs = 192.168.99.2/32
{{</ highlight >}}

Replace `br0` with the network interface that connects to your LAN. In most
cases, this will be `eth0` or something along the lines of `enp1s0`. Use the
`ip a` command to determine this.

Next, add in the SERVER private key, and the public key for peer1 into this
file.

## Creating a configuration file for the client

This file will work as a configuration for the client:

{{< highlight ini >}}
[Interface]
PrivateKey = Peer1 private key
Address = 192.168.99.2/32 
 
[Peer]
PublicKey = SERVER public key
AllowedIPs = 192.168.99.0/24, 192.168.1.0/24
Endpoint = SERVER:51820
{{</ highlight >}}

Add in the private key for the client (Peer1) under Interface, and the public
key of the server under Peer.

Next, you'll need to add in the public IP address, or if the IP address is
dynamic, a hostname that utilises dynamic DNS. This will go under the endpoint.
Leave 51820 as-is unless you have changed it, as this is the port WireGuard
will use.

The AllowedIPs configuration option will allow Peer1 to access both the LAN
and VPN (192.168.99.0/24) networks.

The WireGuard interface will listen on UDP port 51820. Remember to port forward
this in your router settings if your server is behind NAT, and allow the port
through your firewall if you are running one.

## Starting WireGuard

Now that you have both the client and server configurations set up, you can now
start WireGuard on each peer.

On the server, I'll use `wg-quick` to quickly bring up the interface, and then
run the `wg` command to verify it is running:

{{< highlight bash >}}
wg-quick up /etc/wireguard/wg0.conf
wg
{{</ highlight >}}

To make the WireGuard interface start upon boot, you can enable the systemd
service for it:

{{< highlight bash >}}
systemctl enable wg-quick@wg0
{{</ highlight >}}

That's it! You should now be able to access your local network resources over
a VPN. You can test connectivity by pinging the server peer address
(192.168.99.1), and a LAN address such as 192.168.1.1. If you get a reply from
both addresses, everything is working as it should.
