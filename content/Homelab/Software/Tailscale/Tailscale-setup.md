---
title: Tailscale Setup on OpenWrt
description: Joining the router, laptop, and phone to a Tailscale mesh, and fixing the interface/firewall config needed for OpenWrt to actually respond.
tags:
  - Homelab
  - Tailscale
  - Networking
---

## Overview

This page covers joining the router, laptop, and phone to a Tailscale mesh, and fixing the interface/firewall configuration needed for OpenWrt to respond.

## Motivation

Tailscale provides access to home lab services from outside the building without asking the building's IT department to open any inbound ports.

## Client Setup

- **Phone / laptop:** Install the Tailscale app/package and sign in
- **OpenWrt:** Install Tailscale via `apk update && apk add tailscale`, then run `tailscale up` and authenticate via the printed URL

All three devices appeared in the Tailscale dashboard, but pinging the router over Tailscale initially returned `Destination Port Unreachable`.

## Fixing OpenWrt's Tailscale Networking

Two pieces were missing from the official guide:

1. **New interface:** `Network → Interfaces → Add new interface` named `tailscale`, protocol `Unmanaged`, device `tailscale0`
2. **New firewall zone:** Covering that interface, with `Input: ACCEPT`, `Output: ACCEPT`, `Forward: reject`, `Masquerading: on`, `MSS Clamping: on`, and forwarding allowed to/from the `LAN` zone

After saving, connectivity still didn't work until the Tailscale service itself was restarted. After that, the router became reachable over Tailscale from a phone while switching from Wi-Fi to mobile data.

See [successful ping screenshot](Images/tailscale-ping-openwrt-switching-wifi-to-mobile-data.jpg) for successful pings to the router over Tailscale while the phone switches from the building's Wi-Fi to mobile data.

## Reaching the Rest of the LAN

Being able to reach only the router over Tailscale isn't useful — OpenMediaVault and Proxmox aren't on the mesh themselves. See [[Homelab/Software/Tailscale/Mesh-and-subnet-routing|Mesh and subnet routing]] for how the whole LAN subnet is advertised instead of joining every device individually.

## Account Note

Tailscale requires SSO (no plain email/password signup), meaning account security depends entirely on whatever identity provider was used for that SSO login.
