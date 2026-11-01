---
title: Tailscale Mesh and Subnet Routing
description: Advertising the home lab's LAN subnet through the router so every device is reachable over Tailscale, and getting DNS to follow along.
tags:
  - Homelab
  - Tailscale
  - Networking
---

## Overview

This page covers advertising the home lab's LAN subnet through the router so every device is reachable over Tailscale, and getting DNS to follow along.

## Advertising the LAN as a Subnet Route

Rather than installing Tailscale on every home lab device, the router advertises the entire LAN as a subnet route:

```
tailscale up --advertise-routes=10.10.0.0/28 --snat-subnet-routes=false
```

This only needs to run once — it persists across reboots. The advertised route then must be approved once from the Tailscale admin dashboard (`⋮` on the router's row → `Edit route settings…`).

After approval, other mesh devices can reach any LAN IP directly, including OpenMediaVault.

See [subnet route screenshot](Images/tailscale-omv-access-via-subnet-route-mobile-data.jpg) for reaching OMV's web UI from a phone on mobile data, via the router's advertised LAN subnet route.

See [[Homelab/Logical/Home-network/Addressing-plan|the addressing plan]] for the `10.10.0.0/28` subnet itself.

## Getting DNS to Follow over Tailscale

Even with routing working, devices connected over Tailscale weren't resolving the home lab's custom hostnames. Two separate things had to be fixed:

1. **OpenWrt's DNS server only listened on the LAN interface** — It had to be made to also listen on the Tailscale interface (see [[Homelab/Software/OpenWrt/Dhcp-and-dns-configuration|DHCP and DNS configuration]])
2. **Tailscale's own DNS override** had to be enabled (`Override DNS servers` set to true in the Tailscale admin dashboard) so connected devices actually use the home lab's DNS server instead of their own default resolver

### Red Herring: Mobile Browser DNS

A mobile browser was using its own built-in DNS resolver, bypassing the OS-level Tailscale DNS setting entirely. This is worth ruling out early if hostnames resolve on some apps but not others.
