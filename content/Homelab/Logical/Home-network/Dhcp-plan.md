---
title: DHCP plan
description: The DHCP pool range within the home lab's addressing plan, and the Limit-vs-End mistake made while setting it.
tags:
  - Homelab
  - Networking
  - Home-network
---
Within the [[Homelab/Logical/Home-network/Addressing-plan|`10.10.0.0/28`]] network, I have DHCP hand out a small range of addresses at `10.10.0.6` - `10.10.0.10/28`, leaving the lower addresses free for static assignments (router, Proxmox host, NAS).

# The Limit/End mix-up

OpenWrt's DHCP server config has a `Limit` field that is a **count** of addresses from `Start`, not an absolute end address. I misread it as an end address, which let a lease land outside the intended pool:

![[omv-dhcp-lease-outside-intended-range.png]]
*A DHCP-assigned address that was technically in-network but outside the pool actually intended.*

Correcting `Limit` to the right count fixed the pool boundary going forward:

![[corrected-dhcp-limit-vs-end-fields.png]]
*The corrected DHCP range after fixing the `Limit`/`End` misunderstanding.*

See [[Homelab/Software/OpenWrt/Dhcp-and-dns-configuration#DHCP server range|DHCP and DNS configuration]] for where this is actually configured, including the device that first surfaced the `Limit`-vs-end-address mixup.
