---
title: Proxmox Quick Fixes
description: Small post-install cleanup items for Proxmox.
tags:
  - Homelab
  - Proxmox
---

## Overview

This page covers small post-installation cleanup items for Proxmox.

## Disable the Enterprise Repositories

A fresh Proxmox install enables two enterprise repositories (`ceph-squid` and `pve`) that require a paid subscription key, and otherwise just produce update warnings. Both were disabled under `pve → Updates → Repositories`, and the `No-Subscription` repository was added in their place so the system can still receive updates.

## Change the Root Password

The default `root@pam` password was changed to something more reasonable from the user menu in the top right of the Proxmox web UI.

## Set the DNS Server

Once the home lab's custom DNS was set up on the router, Proxmox's own DNS setting was updated under `pve → System → DNS` to point at it, so hostnames like `pve.uhhhhh` resolve correctly from other machines on the network.
