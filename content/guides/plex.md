---
title: "Plex Media Server install"
date: 2023-04-29
lastmod: 2023-08-04
draft: false
tags:
  - linux
  - plex
---

How to install Plex Media Server in a VM/container and provide it access to
media files stored on a NAS.


## Install Intel graphics drivers (hardware transcoding)

This is optional, and should only be done if you are using an Intel CPU with
Quick Sync Video, which includes all Core i3/i5/i7 CPUs from Sandy Bridge
(2nd gen) onwards. Some Atom, Celeron, and Pentium CPUs also support this.

[Debian Hardware Video Acceleration guide](https://wiki.debian.org/HardwareVideoAcceleration)

1. Install the drivers and the `vainfo` utility. If running Plex on a Proxmox
LXC container, the drivers will need to be installed on both the host and
container.

```bash
# Intel 8th gen or newer
apt install intel-media-va-driver vainfo

# Intel 7th gen or older
apt install i965-va-driver vainfo
```

2. Reboot the host

3. Verify hardware is accessible in the container by running:  
`ls -l /dev/dri`

```
total 0
drwxr-xr-x 2 root root         80 Aug  5 02:44 by-path
crw-rw---- 1 root video  226,   0 Aug  5 02:44 card0
crw-rw---- 1 root render 226, 128 Aug  5 02:44 renderD128
```

4. Run `vainfo`. Should get an output similar to below:
```
error: XDG_RUNTIME_DIR is invalid or not set in the environment.
error: can't connect to X server!
libva info: VA-API version 1.17.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so
libva info: va_openDriver() returns -1
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so
libva info: Found init function __vaDriverInit_1_8
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.17 (libva 2.12.0)
vainfo: Driver version: Intel i965 driver for Intel(R) Kaby Lake - 2.4.1
vainfo: Supported profile and entrypoints
      VAProfileMPEG2Simple            : VAEntrypointVLD
      VAProfileMPEG2Simple            : VAEntrypointEncSlice
      VAProfileMPEG2Main              : VAEntrypointVLD
      VAProfileMPEG2Main              : VAEntrypointEncSlice
      VAProfileH264ConstrainedBaseline: VAEntrypointVLD
      VAProfileH264ConstrainedBaseline: VAEntrypointEncSlice
      VAProfileH264ConstrainedBaseline: VAEntrypointEncSliceLP
      VAProfileH264Main               : VAEntrypointVLD
      VAProfileH264Main               : VAEntrypointEncSlice
      VAProfileH264Main               : VAEntrypointEncSliceLP
      VAProfileH264High               : VAEntrypointVLD
      VAProfileH264High               : VAEntrypointEncSlice
      VAProfileH264High               : VAEntrypointEncSliceLP
      VAProfileH264MultiviewHigh      : VAEntrypointVLD
      VAProfileH264MultiviewHigh      : VAEntrypointEncSlice
      VAProfileH264StereoHigh         : VAEntrypointVLD
      VAProfileH264StereoHigh         : VAEntrypointEncSlice
      VAProfileVC1Simple              : VAEntrypointVLD
      VAProfileVC1Main                : VAEntrypointVLD
      VAProfileVC1Advanced            : VAEntrypointVLD
      VAProfileNone                   : VAEntrypointVideoProc
      VAProfileJPEGBaseline           : VAEntrypointVLD
      VAProfileJPEGBaseline           : VAEntrypointEncPicture
      VAProfileVP8Version0_3          : VAEntrypointVLD
      VAProfileHEVCMain               : VAEntrypointVLD
      VAProfileHEVCMain10             : VAEntrypointVLD
      VAProfileVP9Profile0            : VAEntrypointVLD
      VAProfileVP9Profile2            : VAEntrypointVLD
```

## Install Plex

1. Install gnupg and curl so we can add the Plex repo

```bash
apt install gnupg curl
```

2. Add Plex repo, and add their pgp key

```bash
echo deb https://downloads.plex.tv/repo/deb public main | sudo tee /etc/apt/sources.list.d/plexmediaserver.list
curl https://downloads.plex.tv/plex-keys/PlexSign.key | sudo apt-key add -
```

3. Update repos and install Plex Media Server

```bash
apt update
apt install plexmediaserver
```

4. Give Plex rights to the `render` and `video` groups:

```bash
usermod -aG render plex
usermod -aG video plex
```

## Samba Share

1. Install `cifs-utils`, which is necessary to mount a CIFS/SMB share on a Linux host

```bash
apt install cifs-utils
```

2. Create a group that Plex and your user can use to access the file share

```bash
groupadd mediausers
usermod -aG mediausers myuser
usermod -aG mediausers plex
```

3. Create a Samba credentials file `/etc/smbcredentials`

```
user=myuser
password=mypassword
```

4. Secure the file, by ensuring it is owned by and only accessible by root

```bash
chown root.root /etc/smbcredentials
chmod 600 /etc/smbcredentials
```

5. Create a systemd unit file (`/etc/systemd/system/mnt-media.mount`) to
automount the share on boot:

```ini
[Unit]
Description=Mount media share on boot

[Mount]
What=//nas.local/media
Where=/mnt/media
Options=_netdev,credentials=/etc/smbcredentials,iocharset=utf8,rw,gid=mediausers,file_mode=0664,dir_mode=0775
Type=cifs
TimeoutSec=30

[Install]
WantedBy=multi-user.target
```

This systemd unit file will mount the SMB share `\\nas.local\media` at
`/mnt/media`, giving members of the mediausers group read/write access to the
media files.


6. Start and enable the unit file

```bash
systemctl daemon-reload # if you've made changes to the unit file after running systemctl commands
systemctl start mnt-media.mount
systemctl enable mnt-media.mount
```

Both the plex user and yours should be able to read and write to the
`/mnt/media` mountpoint.

## Migrating Plex database to another host

On the new host:

```bash
# Stop plex
systemctl stop plexmediaserver.service

# Backup old database
mv /var/lib/plexmediaserver/ /var/lib/plexmediaserver.old

# Transfer database from old host to the new one
rsync -av colin@192.168.1.4:/home/colin/docker/plex/config/ /var/lib/plexmediaserver/

# Change ownership of the database
chown -R plex.plex /var/lib/plexmediaserver/

# Start Plex
systemctl start plexmediaserver.service
```

Remember to adjust the location of your library's media files if the location
has changed.

---

## Hardware Encoding Troubleshooting

1. Ensure the correct Intel [video drivers](#install-intel-graphics-drivers-hardware-transcoding) are installed.

2. Sometimes the `renderD128` device shows up as a group other than render. If this is the case, change the group back to render by running:  
`chgrp render /dev/dri/renderD128`

3. Reinstall Plex by running `apt reinstall plexmediaserver`. Once it has finished
reinstalling, look in the console output for a line that says **Intel i915 Hardware**. It should say **Found**.

```
PlexMediaServer install:   Processor:           Intel(R) Core(TM) i7-7700T CPU @ 2.90GHz
PlexMediaServer install:   Intel i915 Hardware: Found
PlexMediaServer install:   Nvidia GPU card:     Not Found
```
