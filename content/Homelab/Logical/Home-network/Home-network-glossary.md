---
title: Home Network Glossary
description: Terms used across the Home-network logical bucket.
tags:
  - glossary
---
## Subnet mask / CIDR prefix

The number after the slash in an address like `10.10.0.0/28` — it determines how many addresses a network block contains and where the boundary between the network portion and the host portion of an address falls. A `/24` network holds 254 usable addresses; a `/28` holds 14; a `/29` holds 6. Smaller subnets mean less broadcast traffic per device (every device on a subnet has to process every broadcast sent on it), at the cost of fewer usable addresses. I picked `/28` for my [[Homelab/Logical/Home-network/Addressing-plan|addressing plan]] specifically because it's small enough to keep broadcast traffic down but still leaves room to add a few more devices later without re-planning the whole network. See [Subnetting (Wikipedia)](https://en.wikipedia.org/wiki/Subnetwork) and [why subnetting matters](https://serveravatar.com/what-is-subnetting-and-why-it-matters/#why-subnetting-matters-1).

## DHCP

A protocol that automatically assigns IP addresses (and other network settings, like the default gateway and DNS server) to devices as they join a network, drawing from a defined address pool, so I don't have to manually configure an IP on every laptop and phone that joins the LAN. The pool/range definition lives on whichever device runs the DHCP *server* (here, [[Homelab/Software/OpenWrt/Dhcp-and-dns-configuration|OpenWrt]], running on my router); the overall addressing scheme that pool is drawn from — which addresses are reserved for static assignments versus handed out dynamically — is this Home-network bucket's concern. See my [[Homelab/Logical/Home-network/Dhcp-plan|DHCP plan]] for the actual pool range I use and a mistake I made configuring it.

## Fully qualified domain name (FQDN) / TLD

A complete domain name including its top-level domain (TLD) suffix — e.g. the `.uhhhhh` in `omv.uhhhhh`. Every hostname I assign in my [[Homelab/Logical/Home-network/Dns-naming-scheme|DNS naming scheme]] is really an FQDN once the TLD is attached, and it's the full FQDN (not just the short hostname) that has to match on both the DNS record and the TLS certificate for HTTPS to work without a browser warning. See [FQDN (Wikipedia)](https://en.wikipedia.org/wiki/Fully_qualified_domain_name).
