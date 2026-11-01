---
title: OpenWrt Overview
description: What OpenWrt is used for in this home lab, and which services run on it.
tags:
  - Homelab
  - OpenWrt
---

## Overview

OpenWrt is the router OS running on the Lenovo ThinkPad P52s. Besides routing, it hosts the home lab's DNS, reverse proxy, and mesh-VPN services.

See [router services diagram](Images/openwrt-router-services-diagram.png) for the services planned to run on the OpenWrt router: DNS, Nginx, and Tailscale.

See [full home lab services layout](Images/homelab-services-layout-diagram.png) for the broader plan for every service across the whole home lab at the time of hardware assembly, grouping services onto VMs so related services can be managed and networked together.

## Pages in This Bucket

- [[Homelab/Software/OpenWrt/Installation/Pt 1, Choosing-the-router-os|Installation]] — Sequential walkthrough of picking OpenWrt and getting it running on hardware
- [[Homelab/Software/OpenWrt/Wifi-driver-installation|Wi-Fi driver installation]]
- [[Homelab/Software/OpenWrt/Enterprise-wifi-authentication|Enterprise Wi-Fi authentication]]
- [[Homelab/Software/OpenWrt/Grub-boot-timeout|GRUB boot timeout]]
- [[Homelab/Software/OpenWrt/Ssh-hardening|SSH hardening]]
- [[Homelab/Software/OpenWrt/System-maintenance|System maintenance]]
- [[Homelab/Software/OpenWrt/Dhcp-and-dns-configuration|DHCP and DNS configuration]]
