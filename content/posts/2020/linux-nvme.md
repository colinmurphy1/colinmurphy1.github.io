---
title: "Booting Linux off an NVMe SSD on an older computer"
date: 2020-09-07T21:06:17-05:00
draft: false
tags: ['linux', 'debian']
keywords: ['linux', 'nvme ssd', 'debian nvme', 'linux nvme', 'booting linux off nvme']
description: "How to boot Linux off an NVMe SSD on an unsupported computer."
authors: Colin
aliases:
  - /posts/2020/09/booting-linux-off-an-nvme-ssd-on-an-older-computer/
  - /posts/linux-nvme/
---

Most pre-2015 computers, like my HP Z230 workstation, are only able to boot off of SATA disks. While SATA SSDs are perfectly adequate for most workloads, I wanted to install my operating system on a much faster PCIe-based NVMe SSD.

Despite being able to boot from SATA disks, you can effectively trick an older computer into loading an Linux off a NVMe SSD by installing the GRUB bootloader and boot partition on a SATA SSD or hard drive.

<!--more-->

Here is how I did this on Debian:

First, on the SATA SSD, I created 2 partitions: a **100MB EFI System Partition**, and a **250MB `/boot` partition**.

The Debian installer will install the GRUB bootloader on this disk, as it is the one that contains the EFI system partition. The `/boot` partition MUST be on this disk, otherwise the operating system will be unbootable.

Next, I created a root partition and my other partitions (`/home`, etc) on the NVMe SSD. Keep in mind that you can also use your SATA disk for other partitions as well. In my case, I created a swap partition and kept the rest of the disk empty.

For reference, my final partition layout looks like this:

    colin@z230:~$ lsblk
    NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sdb           8:16   0 111.8G  0 disk
    ├─sdb1        8:17   0    94M  0 part /boot/efi
    ├─sdb2        8:18   0   238M  0 part /boot
    └─sdb3        8:19   0   7.5G  0 part [SWAP]
    nvme0n1     259:0    0 238.5G  0 disk
    ├─nvme0n1p1 259:1    0  59.6G  0 part /
    └─nvme0n1p2 259:2    0 178.9G  0 part /home

That is all you need to do. When the installation is complete, you should see the bootloader screen like you would on a traditional install. 