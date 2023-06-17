---
title: "Plex Media Server install"
date: 2023-04-29
draft: false
tags:
  - linux
  - plex
---

How to install Plex Media Server in a VM/container and provide it access to
media files stored on a NAS.

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
