---
title: "About this website"
description: "About me and this website"
date: 2024-07-15
cover: "/img/miata.jpg"
---

Welcome to my website! This website contains blog posts pertaining to linux,
windows, networking, programming, and other things I may find interesting or
useful. In the future I also plan on posting travel photos on here as I travel
to new places around the United States.

<!--more-->

I currently work as a systems engineer in the higher education sector. Most of
my day to day work revolves around supporting Windows and Microsoft 365
infrastructure, but I do get to and enjoy using Linux from time to time.

## How this website is built

This website is built using the excellent [Hugo][hugo] static site generator,
and a [custom-built theme][theme]. I write all posts, notes, and pages using
Markdown in Visual Studio Code. Other tools used with this website include:

- [Cloudflare](https://cloudflare.com) - DNS and CDN
- [Goatcounter](https://goatcounter.com) - Privacy-friendly web analytics
- [Azure Static Web Apps][azure] - Web Hosting

## Other cool stuff

### The home server

{{< figure
  src="/img/server.jpg"
  alt="The home server"
  caption="The home server in all of its glory"
  position="center" >}}


I run a few services off a custom-built server I built mainly out of parts from
an old storage appliance. Specs are below:

* Intel Xeon E5-2618L v4 CPU
* Supermicro X10SRi-F motherboard
* 256GB DDR4 ECC Registered RAM
* Noctua NH-U9DX i4 cooler
* 500GB NVMe boot drive
* 4x 4TB WD HC310 drives in a ZFS 2-way mirror
* Proxmox VE hypervisor

## Contact me

If you have any comments on the site, or just want to say hi, you can reach out
to me via email. My email address is colin@colinmurphy.me.

[hugo]:https://gohugo.io
[theme]:https://github.com/colinmurphy1/hugo-theme-tux
[azure]:https://azure.microsoft.com/en-us/products/app-service/static
