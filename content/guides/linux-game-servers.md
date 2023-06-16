---
title: "Linux Game Servers"
date: 2023-06-16
draft: false
description: "Linux game servers"
tags:
  - linux
  - gaming
  - minecraft
  - beammp
---

A quick guide on installing a Minecraft and/or BeamMP server on Debian.

As I write guides for installing different game servers, this page may be split
up into multiple pages.

## Minecraft Server

Create a directory for the Minecraft server

```bash
mkdir -p ~/games/minecraft
cd ~/games/minecraft
```

Install Java JRE Headless

```bash
sudo apt install default-jre-headless
```

Download latest Minecraft from https://www.minecraft.net/en-us/download/server

```bash
wget https://piston-data.mojang.com/v1/objects/84194a2f286ef7c14ed7ce0090dba59902951553/server.jar
```

Make systemd service directory (as minecraft user) and make a new service file:
```bash
mkdir -p ~/.config/systemd/user
nano ~/.config/systemd/user/minecraft.service
```

Contents of file `~/.config/systemd/user/minecraft.service`:

```ini
[Unit]
Description=Minecraft Server

[Service]
WorkingDirectory=%h/games/minecraft
ExecStart=java -Xms1G -Xmx2G -jar server.jar

[Install]
WantedBy=multi-user.target
```

Start and enable service

```bash
systemctl --user daemon-reload
systemctl --user start minecraft
systemctl --user enable minecraft
```

Open port in firewall

```bash
sudo firewall-cmd --permanent --zone=public --add-port=30814/tcp 
```

## BeamMP Server


Create game server folder and install dependencies (including dtach)

```bash
mkdir -p ~/games/beammp
cd ~/games/beammp
sudo apt install liblua5.3-0 libssl3 curl dtach
```

Download BeamMP from [GitHub](https://github.com/BeamMP/BeamMP-Server/releases).

```bash
# Even though this is on Debian 12, I'm using the ubuntu 22.04 binary because it just werks
wget https://github.com/BeamMP/BeamMP-Server/releases/download/v3.1.1/BeamMP-Server-ubuntu-22.04 -O beammp-server
```

Configure BeamMP using the directions on the [BeamMP Wiki](https://wiki.beammp.com/en/home/server-installation).

Create systemd service file at `~/.config/systemd/user/beammp.service`. Contents:

```ini
[Unit]
Description=BeamMP Server

[Service]
Type=forking
Restart=on-failure
RestartSec=5
WorkingDirectory=%h/games/beammp/
ExecStart=dtach -n /tmp/beammp ./beammp-server

[Install]
WantedBy=multi-user.target
```

Start and optionally enable service
```bash
systemctl --user daemon-reload
systemctl --user start beammp
systemctl --user enable beammp
```

Open port UDP and TCP port 30814, the port the BeamMP server listens on

```bash
sudo firewall-cmd --permanent --zone=public --add-port=30814/tcp --add-port=30814/udp
```

If you need to use the BeamMP server console, run the following command:

```bash
dtach -a /tmp/beammp
```

You can detach from BeamMP console by hitting <kbd>Ctrl + \\</kbd>. If you
accidentally kill the server, it will automatically restart in 5 seconds.
