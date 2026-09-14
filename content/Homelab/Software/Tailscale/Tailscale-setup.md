---
title: Tailscale Setup
description: Joining the router, a laptop, and a phone to a Tailscale mesh, and fixing the interface/firewall config needed for OpenWrt to actually respond.
tags:
  - Homelab
  - Tailscale
  - Networking
---
# Why

I use [[Homelab/Software/Tailscale/Tailscale-glossary#Tailscale|Tailscale]] to reach home lab services from outside the building without asking the building's IT department to open any inbound ports.

# Client setup

- **Phone / laptop:** I installed the Tailscale app/package and signed in — done.
- **OpenWrt:** OpenWrt doesn't ship Tailscale by default, but has an [official Tailscale guide](https://openwrt.org/docs/guide-user/services/vpn/tailscale/start). I installed it via `apk update && apk add tailscale`, then ran `tailscale up` and authenticated via the URL it printed.

All three devices appeared in the Tailscale dashboard, but pinging the router over Tailscale initially returned `Destination Port Unreachable`.

# Fixing OpenWrt's Tailscale networking

I was still missing two pieces, both from the official guide:

1. A new **interface** (`Network → Interfaces → Add new interface`) named `tailscale`, protocol `Unmanaged`, device `tailscale0`.
2. A new **firewall zone** (`Network → Firewall → Zones → Add`) covering that interface, with `Input: ACCEPT`, `Output: ACCEPT`, `Forward: reject`, `Masquerading: on`, `MSS Clamping: on`, and forwarding allowed to/from the `LAN` zone.

After saving, connectivity still didn't work until I restarted the Tailscale service itself — after that, I could reach the router over Tailscale from my phone while switching from Wi-Fi to mobile data:

![[tailscale-ping-openwrt-switching-wifi-to-mobile-data.jpg]]
*Successful pings to the router over Tailscale while the phone switches from the building's Wi-Fi to mobile data.*

# Reaching the rest of the LAN, not just the router

Being able to reach only the router over Tailscale isn't very useful to me — [[Homelab/Software/OpenMediaVault/Openmediavault-installation|OMV]] and [[Homelab/Software/Proxmox/Proxmox-installation|Proxmox]] aren't on the mesh themselves. See [[Homelab/Software/Tailscale/Mesh-and-subnet-routing|Mesh and subnet routing]] for how I advertise the whole LAN subnet instead of joining every device individually.

# Account note

Tailscale requires SSO (no plain email/password signup), meaning my account security here depends entirely on whatever identity provider I used for that SSO login.
