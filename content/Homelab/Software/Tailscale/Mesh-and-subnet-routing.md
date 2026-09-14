---
title: Mesh and subnet routing
description: Advertising the home lab's LAN subnet through the router so every device is reachable over Tailscale, and getting DNS to follow along.
tags:
  - Homelab
  - Tailscale
  - Networking
---
# Advertising the LAN as a subnet route

Rather than installing Tailscale on every home lab device, I have the router advertise the entire LAN as a [[Homelab/Software/Tailscale/Tailscale-glossary#Subnet router|subnet route]]:

```
tailscale up --advertise-routes=10.10.0.0/28 --snat-subnet-routes=false
```

I only need to run this once — it persists across reboots. I then had to approve the advertised route once from the Tailscale admin dashboard (`⋮` on the router's row → `Edit route settings…`). After approval, other mesh devices can reach any LAN IP directly, including [[Homelab/Software/OpenMediaVault/Openmediavault-installation|OMV]]:

![[tailscale-omv-access-via-subnet-route-mobile-data.jpg]]
*Reaching OMV's web UI from a phone on mobile data, via the router's advertised LAN subnet route.*

See [[Homelab/Logical/Home-network/Addressing-plan|the addressing plan]] for the `10.10.0.0/28` subnet itself.

# Getting DNS to follow over Tailscale

Even with routing working, I found that devices connected over Tailscale weren't resolving [[Homelab/Logical/Home-network/Dns-naming-scheme|the home lab's custom hostnames]]. I had to fix two separate things:

1. **OpenWrt's DNS server only listened on the LAN interface** — I had to make it also listen on the Tailscale interface (see [[Homelab/Software/OpenWrt/Dhcp-and-dns-configuration|DHCP and DNS configuration]]).
2. **Tailscale's own DNS override** had to be enabled (`Override DNS servers` set to true in the Tailscale admin dashboard) so connected devices actually use the home lab's DNS server instead of their own default resolver.

While debugging this, I ran into a red herring: a mobile browser was using its own built-in DNS resolver, bypassing the OS-level Tailscale DNS setting entirely. Worth ruling out early if hostnames resolve on some apps but not others.
