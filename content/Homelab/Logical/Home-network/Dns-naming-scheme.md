---
title: DNS Naming Scheme on OpenWrt Router
description: The home lab's custom top-level domain and per-device hostnames.
tags:
  - Homelab
  - Networking
  - Home-network
  - DNS
---

## Overview

This page documents the home lab's custom top-level domain and per-device hostnames.

## Custom TLD

Rather than a generic TLD like `.com` or `.net`, a custom, unregistered TLD is used for the home lab: `.uhhhhh`.

### Domain Configuration

Setting the router's `Local domain` field alone doesn't retroactively migrate hostnames already recorded under a different suffix. The target domain (including the TLD) must be set explicitly on each hostname record to move it over.

See [custom TLD screenshot](Images/omv-accessible-via-custom-tld-uhhhhh.png) for the NAS reachable at `omv.uhhhhh` after explicitly setting the domain on its DNS record.

### HTTPS Certificates

Because this TLD isn't a registered public domain, only a private certificate authority can issue trusted HTTPS certificates for it. See [[Homelab/Software/Nginx/Ssl-certificates|SSL certificates]].

## Hostnames

Custom hostname records are added on the router, mapping each home lab device to a friendly name under `.uhhhhh`, e.g., `omv.uhhhhh` for the NAS and `pve.uhhhhh` for the Proxmox host (fronted by Nginx so the port doesn't need to be typed).

See [[Homelab/Software/OpenWrt/Dhcp-and-dns-configuration|DHCP and DNS configuration]] for where these records are configured.
