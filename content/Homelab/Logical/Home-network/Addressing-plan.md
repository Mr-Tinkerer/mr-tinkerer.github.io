---
title: Home Network Addressing Plan
description: The home lab's network address, subnet size, and per-device static IP assignments.
tags:
  - Homelab
  - Networking
  - Home-network
---

## Overview

This page documents the home lab's network address, subnet size, and per-device static IP assignments.

## Layout

With only one router and one switch in the home lab, a traditional multi-tier network topology wasn't needed. The overall services/subnet layout was planned ahead of deployment.

See [network topology diagram](Images/homelab-network-topology-diagram.png) for the home lab's planned network topology and subnet usage.

## Subnet Size

A `/28` subnet mask was chosen over the default `/24`:

- A `/24` allows 254 devices, far more than this home lab needs
- Every extra device on the subnet adds broadcast traffic
- A `/29` (6 devices) would be even more efficient
- `/28`'s 14 addresses leave room to grow without needing to re-plan the network again soon

## Network Address

`10.10.0.0/28` was chosen as the network address, rather than the more common `192.168.1.0/24` or `10.0.0.0/24`:

- Both of those ranges are common defaults that are more likely to collide with other networks reached over a VPN (e.g., Tailscale)
- Picking a less common address inside the `10.0.0.0/8` private range reduces the odds of an overlapping-subnet conflict when connecting to other networks

## Static Assignments

| Device | Address |
|--------|---------|
| Router (ThinkPad / OpenWrt) LAN | `10.10.0.14/28` |
| Proxmox host (`vmbr0`) | `10.10.0.12` |
| NAS (OpenMediaVault) | `10.10.0.13` |
| DHCP pool (other clients) | starts at `10.10.0.6` - ends at `10.10.0.10` |

Windows and other DHCP clients pick up addresses from the low end of the pool automatically (the first Windows 11 test picked up `10.10.0.6`).
