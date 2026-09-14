---
title: DHCP and DNS configuration
description: Configuring OpenWrt's dnsmasq-based DHCP server and custom DNS records to serve the home lab's addressing plan.
tags:
  - Homelab
  - OpenWrt
  - DNS
---
This page covers the *mechanics* of configuring OpenWrt's DHCP/DNS server (dnsmasq under the hood). For the addressing scheme itself — the subnet, the network address, and which device gets which IP — see [[Homelab/Logical/Home-network/Addressing-plan|the Home-network addressing plan]] (a DHCP *server* is software; the addressing *plan* it serves is a logical scheme spanning every device on the network).

# Changing the router's own LAN address

When I edited the LAN interface's address under `Network → Interfaces` and saved/applied it, it immediately dropped any client currently on the old subnet, including the machine I was configuring from. OpenWrt has a safety feature for this: it auto-reverts a LAN address change if the new address isn't reached within roughly 90 seconds, to avoid permanently locking the admin out.

![[openwrt-lan-ip-change-pings-dropping-auto-revert.png]]
*Pings dropping partway through — this is OpenWrt auto-reverting the LAN address change because the new address wasn't reached in time.*

**Consequence:** I had to reconfigure the client machine to use the new subnet *before* the 90-second window closed, or the router would silently revert to the old address and I'd have to redo the change.

I also found a router can temporarily hold more than one IP/subnet on the same interface at once (e.g. keeping the old `192.168.1.1/24` alongside a new `10.10.0.14/28`), which is useful for reaching other devices still on the old subnet while migrating them one at a time — OpenWrt automatically adds a route for each address it holds.

![[multiple-ip-addresses-routing-both-subnets.png]]
*Successfully reaching a device still on the old `192.168.1.0/24` subnet from a client already migrated to the new `10.10.0.0/28` subnet, thanks to the router holding both addresses.*
# DHCP server range

Under `Network → Interfaces → [LAN] → DHCP Server → IPv4 Settings`, the **Start** and **Limit** fields define the DHCP pool — note that `Limit` is a *count* of addresses from `Start`, not an absolute end address; I misread it as the end address once, and it handed out leases outside the intended range. See [[Homelab/Logical/Home-network/Dhcp-plan|the DHCP plan]] for the specific ranges used.

This is exactly what happened once I applied [[Homelab/Logical/Home-network/Addressing-plan|the home lab's addressing scheme]]: the [[Homelab/Hardware/Dell-Vostro-260s/Dell-vostro-260s-overview|NAS]] picked up a DHCP lease that was technically within the correct network but outside the range I meant to hand out:

![[omv-dhcp-lease-outside-intended-range.png]]
*The NAS's DHCP-assigned address, unexpectedly outside the intended pool.*

**Cause:** I had misread the `Limit` field as an absolute *end* address instead of a *count* of addresses from `Start`. Correcting `Limit` to the right count fixed the pool boundary:

![[corrected-dhcp-limit-vs-end-fields.png]]
*The corrected DHCP range after fixing the `Limit`/`End` misunderstanding.*

# Custom DNS hostnames and TLD

I added custom hostname → IP records under `Network → DNS → Hostnames`. The `Local domain` field under `Network → DNS → General` sets the suffix appended to hostnames served over DHCP; I found that setting it alone doesn't retroactively rename hostnames already configured with a different domain suffix — each hostname record needs the desired domain included explicitly.

By default, dnsmasq only listens for DNS queries on the LAN interface. To make it also resolve for clients connected over [[Homelab/Software/Tailscale/Tailscale-setup|Tailscale]], I had to add the Tailscale interface to its listening interfaces alongside LAN. See [[Homelab/Logical/Home-network/Dns-naming-scheme|the DNS naming scheme]] for the actual hostnames and TLD chosen.
