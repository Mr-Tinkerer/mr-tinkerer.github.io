---
title: Tailscale Glossary
description: Terms used across the Tailscale bucket.
tags:
  - glossary
---

## Overview

This page documents terms used across the Tailscale bucket.

## Tailscale

A mesh VPN service built on WireGuard, creating peer-to-peer (or relayed, when direct connections aren't possible) encrypted links between devices without requiring port forwarding. It's used so home lab services can be reached from outside the building without asking the building's IT department to open any inbound ports on their network — each device joined (router, laptop, phone) just needs the Tailscale app/package and a login, and it shows up on the mesh automatically. See [[Homelab/Software/Tailscale/Tailscale-setup|Setup]] for how the router itself was joined, which was the tricky part. See [Tailscale (Wikipedia)](https://en.wikipedia.org/wiki/Tailscale).

## Subnet Router

A Tailscale node that advertises an entire local subnet to the mesh, so other mesh devices can reach machines on that subnet without each of them needing Tailscale installed individually. The OpenWrt router acts as the subnet router for the home lab's whole LAN, which lets devices like OMV be reached over Tailscale even though OMV itself never joined the mesh — see [[Homelab/Software/Tailscale/Mesh-and-subnet-routing|Mesh and subnet routing]] for how this was set up and the route approved. See [Tailscale subnet routers (official docs)](https://tailscale.com/kb/1019/subnets).
