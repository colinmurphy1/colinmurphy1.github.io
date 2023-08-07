---
title: "Debian Installation Guide"
date: 2023-06-16
draft: false
description: "Debian Linux install steps"
tags:
  - linux
  - debian
---

This is a quick guide on the process I go through after installing Debian. These
steps have been tested on Debian 12 (bookworm), but are likely to work on other
releases of Debian.

1. Log in as your normal (non-root) user.

2. Update the repos, install updates, and install some common utilities:

```bash
su -
# enter root password
apt update
apt -y dist-upgrade
apt -y install sudo htop nano vim tmux dtach firewalld fail2ban rsync curl git
```

3. Copy SSH key as your **normal user**.

```bash
mkdir .ssh
echo "your-pubkey" > .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
chmod 700 .ssh/
```

**Note:** This can be done using `ssh-copy-id`. However, this utility is absent
from Windows, which is my daily driver OS.

4. Disconnect from server and then reconnect. It should automatically sign you
in without a password prompt.

5. Add your non-root user to the sudo group, which allows you to use `sudo` to
gain root access.

```bash
su -
usermod -aG sudo your-user-name
```

Log out, and then log back in. Test sudo by running `sudo -i`. You should get
a root shell.

6. Disable password logins with ssh

```bash
# do this as root
nano /etc/ssh/sshd_config
```

Make the following changes:

- Set **PasswordAuthentication** to **no**
- Add **AuthenticationMethods** and set it to **publickey**

7. Restart sshd

```bash
systemctl restart sshd
```

8. Set locale to the locale of your country, in my case `en_US.UTF-8`. This is
not necessary on regular Debian installations, but I have found this necessary
when using Linux containers in Proxmox.

```bash
dpkg-reconfigure locales
```

9. Configure firewall

```bash
# Start and enable the firewalld service
systemctl start firewalld
systemctl enable firewalld

# Add eno1 interface to the public zone - find interface first with `ip a`
firewall-cmd --zone=public --change-interface=eno1
```

After adding an interface, the public zone will now become the default zone in
firewalld.

This should have already been done, but allow SSH traffic:

```bash
firewall-cmd --permanent --zone=public --add-service=ssh
```

Additional services or ports can be enabled like so:

```bash
# Open HTTP and HTTPS ports
firewall-cmd --permanent --zone=public --add-service={http,https}

# Open Minecraft port
firewall-cmd --permanent --zone=public --add-port=25565/tcp
```

Verify changes to firewall, once done:

```bash
firewall-cmd --list-all
```

[Firewalld beginners guide](https://www.redhat.com/sysadmin/beginners-guide-firewalld)

10. Configure fail2ban to block SSH bruteforce login attempts

WIP
