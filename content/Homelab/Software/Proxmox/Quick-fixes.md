---
title: Quick fixes
description: Small post-install cleanup items for Proxmox.
tags:
  - Homelab
  - Proxmox
---
# Disable the enterprise repositories

A fresh Proxmox install enables two [[Homelab/Software/Proxmox/Proxmox-glossary#Enterprise repository|enterprise repositories]] (`ceph-squid` and `pve`) that require a paid subscription key, and otherwise just produce update warnings. I disabled both under `pve → Updates → Repositories`, and added the `No-Subscription` repository in their place so the system can still receive updates.

# Change the root password

I changed the default `root@pam` password to something more reasonable from the user menu in the top right of the Proxmox web UI.

# Set the DNS server

Once I'd set up [[Homelab/Logical/Home-network/Dns-naming-scheme|the home lab's custom DNS]] on the router, I updated Proxmox's own DNS setting under `pve → System → DNS` to point at it, so hostnames like `pve.uhhhhh` resolve correctly from other machines on the network.
