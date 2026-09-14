---
title: OpenMediaVault Installation
description: Wiping old partitions and installing OpenMediaVault onto the NAS build.
tags:
  - Homelab
  - OpenMediaVault
---
Before installing, I found the [[Homelab/Hardware/Dell-Vostro-260s/Dell-vostro-260s-overview|NAS hardware's]] drives still had leftover partitions from previous, unrelated OSes (only one of which was clearly labeled). I wiped these by booting a live Arch Linux environment, listing partitions with `lsblk`, and using `fdisk` to write a fresh GPT partition table over each drive.

I booted the installer itself from Ventoy; one boot attempt hit an unexplained graphical glitch that resolved itself after a few minutes with no further recurrence or clear cause:

![[ventoy-graphical-glitch-booting-omv.png]]
*A one-off graphical glitch in Ventoy while booting the OMV installer — booted normally on its own after a short wait; cause unknown.*

After the OMV installer completed, I could reach its web UI normally:

![[omv-web-interface-loaded-after-install.png]]
*OpenMediaVault's web UI reachable for the first time after install.*

Under `network → Interfaces`, I switched the interface from DHCP to a static address matching [[Homelab/Logical/Home-network/Addressing-plan|the addressing plan]], and pointed its DNS server at the router (which also runs [[Homelab/Software/OpenWrt/Dhcp-and-dns-configuration|the home lab's custom DNS]]).

See [[Homelab/Software/OpenMediaVault/Quick-fixes|Quick fixes]] for post-install cleanup.
