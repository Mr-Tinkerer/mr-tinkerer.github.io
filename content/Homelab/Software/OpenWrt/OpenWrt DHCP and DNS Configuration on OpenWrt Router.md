---
title: OpenWrt DHCP and DNS Configuration on OpenWrt Router
description: Configuring OpenWrt's dnsmasq-based DHCP server and custom DNS records to serve the home lab's addressing plan.
tags:
  - Homelab
  - OpenWrt
  - DNS
---

## Overview

This page covers the mechanics of configuring OpenWrt's DHCP/DNS server (dnsmasq under the hood) to serve the home lab's addressing plan.

For the addressing scheme itself — the subnet, network address, and which device gets which IP — see [[Homelab/Logical/Home-network/Addressing-plan|the Home-network addressing plan]] (a DHCP server is software; the addressing plan it serves is a logical scheme spanning every device on the network).

## Changing the Router's Own LAN Address

When the LAN interface's address is edited under `Network → Interfaces` and saved/applied, it immediately drops any client currently on the old subnet, including the machine being configured from.

### Safety Feature

OpenWrt has a safety feature that auto-reverts a LAN address change if the new address isn't reached within roughly 90 seconds, to avoid permanently locking the admin out.

See [LAN IP change screenshot](Images/openwrt-lan-ip-change-pings-dropping-auto-revert.png) for pings dropping partway through — OpenWrt auto-reverting the LAN address change because the new address wasn't reached in time.

### Consequence

The client machine must be reconfigured to use the new subnet before the 90-second window closes, or the router will silently revert to the old address.

### Multi-IP Support

A router can temporarily hold more than one IP/subnet on the same interface at once (e.g., keeping the old `192.168.1.1/24` alongside a new `10.10.0.14/28`). This is useful for reaching other devices still on the old subnet while migrating them one at a time. OpenWrt automatically adds a route for each address it holds.

See [multi-IP routing screenshot](Images/multiple-ip-addresses-routing-both-subnets.png) for successfully reaching a device still on the old `192.168.1.0/24` subnet from a client already migrated to the new `10.10.0.0/28` subnet, thanks to the router holding both addresses.

## DHCP Server Range

Under `Network → Interfaces → [LAN] → DHCP Server → IPv4 Settings`, the **Start** and **Limit** fields define the DHCP pool.

### Limit Field Misunderstanding

The `Limit` field is a **count** of addresses from `Start`, not an absolute end address.

This was misread as an end address once, causing DHCP leases to be handed out outside the intended range. See [[Homelab/Logical/Home-network/Dhcp-plan|the DHCP plan]] for the specific ranges used.

### Example Failure

The NAS picked up a DHCP lease that was technically within the correct network but outside the range intended to be handed out.

See [DHCP lease outside range screenshot](Images/omv-dhcp-lease-outside-intended-range.png) for the NAS's DHCP-assigned address unexpectedly outside the intended pool.

### Resolution

Correcting `Limit` to the right count fixed the pool boundary.

See [corrected DHCP range screenshot](Images/corrected-dhcp-limit-vs-end-fields.png) for the corrected DHCP range after fixing the `Limit`/`End` misunderstanding.

## Custom DNS Hostnames and TLD

Custom hostname → IP records are added under `Network → DNS → Hostnames`.

### Local Domain Suffix

The `Local domain` field under `Network → DNS → General` sets the suffix appended to hostnames served over DHCP. Setting it alone doesn't retroactively rename hostnames already configured with a different domain suffix — each hostname record needs the desired domain included explicitly.

### Tailscale Interface

By default, dnsmasq only listens for DNS queries on the LAN interface. To make it also resolve for clients connected over Tailscale, the Tailscale interface must be added to its listening interfaces alongside LAN. See [[Homelab/Logical/Home-network/Dns-naming-scheme|the DNS naming scheme]] for the actual hostnames and TLD chosen.
