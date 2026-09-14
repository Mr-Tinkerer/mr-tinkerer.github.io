---
title: Proxmox Installation
description: Installing Proxmox VE onto the Dell Optiplex 5060.
tags:
  - Homelab
  - Proxmox
---
I installed Proxmox onto the [[Homelab/Hardware/Dell-Optiplex-5060/Dell-optiplex-5060-overview|Dell Optiplex 5060]] via the standard Proxmox installer, booted from a Ventoy USB stick.

The only real decision I had to make during install was filesystem choice: I ruled out ZFS and BTRFS since Proxmox only offers them in RAID configurations, and this machine has a single boot drive. Between the remaining options, XFS is best suited to large files while this host mostly stores small VM disk images, so I went with **EXT4**.

After install, I could reach the web UI normally, confirming the install had succeeded.

See [[Homelab/Software/Proxmox/Quick-fixes|Quick fixes]] for post-install cleanup (enterprise repos, password, DNS).
