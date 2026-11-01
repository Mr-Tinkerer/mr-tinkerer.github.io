---
title: DHCP Plan
description: The DHCP pool range within the home lab's addressing plan and the Limit-vs-End configuration mistake.
tags:
  - Homelab
  - Networking
  - Home-network
---

## Overview

This page documents the DHCP pool configuration within the `10.10.0.0/28` network and a configuration error that affected the pool boundaries.

## DHCP Pool Configuration

The DHCP server hands out addresses in the range `10.10.0.6` - `10.10.0.10`, leaving lower addresses free for static assignments (router, Proxmox host, NAS).

## Limit/End Configuration Error

### Problem Description

OpenWrt's DHCP server configuration contains a `Limit` field that specifies the **count** of addresses from `Start`, not an absolute end address.

### Configuration Error

The `Limit` field was misinterpreted as an end address, causing a DHCP lease to land outside the intended pool.

### Resolution

Correcting the `Limit` value to the proper count fixed the pool boundary issue for subsequent leases.

See [[Homelab/Software/OpenWrt/Dhcp-and-dns-configuration#DHCP server range|DHCP and DNS configuration]] for the actual configuration location, including the device that first surfaced the `Limit`-vs-end-address misunderstanding.
