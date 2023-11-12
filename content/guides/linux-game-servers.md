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

A quick guide on installing various game servers on Debian. As I write more 
guides for installing different game servers, this page may be split up into
multiple pages.

## Create a user account

Before creating any game servers, we will need to first create a new user
account to run the game servers as. We will also enable user lingering with
loginctl, to ensure that the systemd services created will always remain
running.

1. Create a new user account

```bash
useradd -m -s /bin/bash games
```

2. Enable [user lingering](lingering) for the games user

```bash
loginctl enable-linger games
```

## Minecraft Server

1. Install Java JRE Headless

```bash
apt install default-jre-headless
```

2. Create a new directory for the Minecraft server

```bash
su games -
mkdir -p ~/games/minecraft
cd ~/games/minecraft
```

3. Download latest Minecraft server .jar file from https://www.minecraft.net/en-us/download/server,
and save it to the `~/games/minecraft` directory.

4. Make systemd service directory (as minecraft user) and make a new service file:

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
WantedBy=default.target
```

5. Start and enable service

```bash
systemctl --user daemon-reload
systemctl --user start minecraft
systemctl --user enable minecraft
```

6. Open port 25565/tcp in firewall to allow others to connect to the server

```bash
firewall-cmd --permanent --zone=public --add-port=25565/tcp 
```

7. Launch Minecraft on your computer and connect to the server

---

## BeamMP Server

1. Install BeamMP dependencies and dtach

```bash
apt install liblua5.3-0 libssl3 curl dtach
```

2. Create game server folder

```bash
mkdir -p ~/games/beammp
cd ~/games/beammp
```

3. Download BeamMP from [GitHub](https://github.com/BeamMP/BeamMP-Server/releases).

**Note:** At the time of writing, there is no Debian 12 binary. Instead, use the
Ubuntu 22.04 binary which appears to work fine.

```bash
wget https://github.com/BeamMP/BeamMP-Server/releases/download/v3.1.1/BeamMP-Server-ubuntu-22.04 -O beammp-server

# make the binary executable
chmod +x beammp-server
```

4. Configure BeamMP using the directions on the [BeamMP Wiki](https://wiki.beammp.com/en/home/server-installation).

5. Create systemd service file at `~/.config/systemd/user/beammp.service`. Contents are below.

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
WantedBy=default.target
```

Start and optionally enable service
```bash
systemctl --user daemon-reload
systemctl --user start beammp
systemctl --user enable beammp
```

6. Open port UDP and TCP port 30814, the port the BeamMP server listens on

```bash
firewall-cmd --permanent --zone=public --add-port=30814/tcp --add-port=30814/udp
```

If you need to use the BeamMP server console, run the following command:

```bash
dtach -a /tmp/beammp
```

You can detach from BeamMP console by hitting <kbd>Ctrl + \\</kbd>. If you
accidentally kill the server, it will automatically restart in 5 seconds.

[lingering]:https://www.freedesktop.org/software/systemd/man/latest/loginctl.html#enable-linger%20USER%E2%80%A6
