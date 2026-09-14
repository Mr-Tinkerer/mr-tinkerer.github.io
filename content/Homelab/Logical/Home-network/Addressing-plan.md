---
title: Addressing plan
description: The home lab's network address, subnet size, and per-device static IP assignments.
tags:
  - Homelab
  - Networking
  - Home-network
---
# Layout

With only one router and one switch in the home lab, I didn't need a traditional multi-tier network topology. I planned out the overall services/subnet layout ahead of deployment:

![[homelab-network-topology-diagram.png]]
*The home lab's planned network topology and subnet usage.*

# Subnet size

I chose a `/28` [[Homelab/Logical/Home-network/Home-network-glossary#Subnet mask / CIDR prefix|subnet mask]] over the default `/24`: a `/24` allows 254 devices, far more than this home lab needs, and every extra device on the subnet adds broadcast traffic. A `/29` (6 devices) would be even more efficient, but `/28`'s 14 addresses leaves me room to grow without needing to re-plan the network again soon.

# Network address

I chose `10.10.0.0/28` as the network address, rather than the more common `192.168.1.0/24` or `10.0.0.0/24`: both of those ranges are common defaults that are more likely to collide with other networks I reach over a VPN (e.g. [[Homelab/Software/Tailscale/Tailscale-setup|Tailscale]]). Picking a less common address inside the `10.0.0.0/8` private range reduces the odds of an overlapping-subnet conflict when I connect to other networks.

# Static assignments

| Device                                                                                                                     | Address                                      |
| -------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- |
| Router ([[Homelab/Hardware/Lenovo-Thinkpad-P52s/Lenovo-thinkpad-p52s-overview|ThinkPad]] / [[Homelab/Software/OpenWrt/Openwrt-overview|OpenWrt]]) LAN | `10.10.0.14/28`                              |
| [[Homelab/Hardware/Dell-Optiplex-5060/Dell-optiplex-5060-overview|Proxmox host]] (`vmbr0`)                                                   | `10.10.0.12`                                 |
| [[Homelab/Hardware/Dell-Vostro-260s/Dell-vostro-260s-overview|NAS]] (OpenMediaVault)                                                       | `10.10.0.13`                                 |
| DHCP pool (other clients)                                                                                                  | starts at `10.10.0.6` - ends at `10.10.0.10` |

Windows and other DHCP clients pick up addresses from the low end of the pool automatically (my first Windows 11 test picked up `10.10.0.6`).
