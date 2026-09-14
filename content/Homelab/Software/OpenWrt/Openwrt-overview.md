---
title: OpenWrt Overview
description: What OpenWrt is used for in this home lab, and which services run on it.
tags:
  - Homelab
  - OpenWrt
---
# Role

[[Homelab/Software/OpenWrt/Openwrt-glossary#OpenWrt|OpenWrt]] is the router OS running on the [[Homelab/Hardware/Lenovo-Thinkpad-P52s/Lenovo-thinkpad-p52s-overview|Lenovo ThinkPad P52s]]. Besides routing, it hosts the home lab's DNS, reverse proxy, and mesh-VPN services:

![[openwrt-router-services-diagram.png]]
*The services planned to run on the.pngnWrt router: DNS, [[Homelab/Software/Nginx/Reverse-proxy-setup|Nginx]], and [[Homelab/Software/Tailscale/Tailscale-setup|Tailscale]].*

Here's the broader plan I had for every service across the whole home lab (not just the router) at the time I was assembling hardware:

![[homelab-services-layout-diagram.png]]
*The full home lab services layout as planned, grouping services onto VMs so related services can be managed and networked together.*

# Pages in this bucket

- [[Homelab/Software/OpenWrt/Installation/Pt 1, Choosing-the-router-os|Installation]] — the sequential walkthrough of picking OpenWrt and getting it running on hardware.
- [[Homelab/Software/OpenWrt/Wifi-driver-installation|Wi-Fi driver installation]]
- [[Homelab/Software/OpenWrt/Enterprise-wifi-authentication|Enterprise Wi-Fi authentication]]
- [[Homelab/Software/OpenWrt/Grub-boot-timeout|GRUB boot timeout]]
- [[Homelab/Software/OpenWrt/Ssh-hardening|SSH hardening]]
- [[Homelab/Software/OpenWrt/System-maintenance|System maintenance]]
- [[Homelab/Software/OpenWrt/Dhcp-and-dns-configuration|DHCP and DNS configuration]]
