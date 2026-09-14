---
title: Tailscale Glossary
description: Terms used across the Tailscale bucket.
tags:
  - glossary
---
## Tailscale

A mesh VPN service built on WireGuard, creating peer-to-peer (or relayed, when direct connections aren't possible) encrypted links between devices without requiring port forwarding. I use it so I can reach home lab services from outside the building without asking the building's IT department to open any inbound ports on their network — each device I join (router, laptop, phone) just needs the Tailscale app/package and a login, and it shows up on the mesh automatically. See [[Homelab/Software/Tailscale/Tailscale-setup|Setup]] for how I joined the router itself, which was the tricky part. See [Tailscale (Wikipedia)](https://en.wikipedia.org/wiki/Tailscale).

## Subnet router

A Tailscale node that advertises an entire local subnet to the mesh, so other mesh devices can reach machines on that subnet without each of them needing Tailscale installed individually. I have the OpenWrt router act as the subnet router for the home lab's whole LAN, which is what lets me reach devices like [[Homelab/Software/OpenMediaVault/Openmediavault-installation|OMV]] over Tailscale even though OMV itself never joined the mesh — see [[Homelab/Software/Tailscale/Mesh-and-subnet-routing|Mesh and subnet routing]] for how I set that up and approved the route. See [Tailscale subnet routers (official docs)](https://tailscale.com/kb/1019/subnets).
