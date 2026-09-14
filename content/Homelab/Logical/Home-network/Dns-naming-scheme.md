---
title: DNS naming scheme
description: The home lab's custom top-level domain and per-device hostnames.
tags:
  - Homelab
  - Networking
  - Home-network
  - DNS
---
# Custom TLD

Rather than a generic [[Homelab/Logical/Home-network/Home-network-glossary#Fully qualified domain name (FQDN) / TLD|TLD]] like `.com` or `.net`, I use a custom, unregistered TLD for the home lab: `.uhhhhh`.

I found that setting the router's `Local domain` field alone doesn't retroactively migrate hostnames already recorded under a different suffix — I had to set the target domain (including the TLD) explicitly on each hostname record to move it over:

![[omv-accessible-via-custom-tld-uhhhhh.png]]
*The NAS reachable at `omv.uhhhhh` after explicitly setting the domain on its DNS record.*

Because this TLD isn't a registered public domain, only a [[Homelab/Software/Nginx/Nginx-glossary#Certificate authority (CA)|private certificate authority]] can issue trusted HTTPS certificates for it — see [[Homelab/Software/Nginx/Ssl-certificates|SSL certificates]].

# Hostnames

I add custom hostname records on the router (see [[Homelab/Software/OpenWrt/Dhcp-and-dns-configuration|DHCP and DNS configuration]]) mapping each home lab device to a friendly name under `.uhhhhh`, e.g. `omv.uhhhhh` for the NAS and `pve.uhhhhh` for the Proxmox host (fronted by [[Homelab/Software/Nginx/Reverse-proxy-setup|Nginx]] so I don't need to type the port).
