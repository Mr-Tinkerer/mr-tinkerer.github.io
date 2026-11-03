---
title: Proxmox Installation on Dell Optiplex 5060
description: Installing Proxmox VE onto the Dell Optiplex 5060.
tags:
  - Homelab
  - Proxmox
---

## Overview

This guide covers Proxmox Virtual Environment (VE) installation on the Dell Optiplex 5060, including boot media preparation and filesystem selection.

## Boot Media Preparation

Install Proxmox onto the Dell Optiplex 5060 via the standard Proxmox installer, booted from a Ventoy USB stick.

## Filesystem Selection

### Decision Process

The filesystem choice required evaluation of available options:

- **ZFS** — Ruled out because Proxmox only offers ZFS in RAID configurations, and this machine has a single boot drive
- **BTRFS** — Ruled out for the same reason (RAID-only availability)
- **EXT4** — Selected as the best option for storing large VM disk images

## Installation Verification

After installation completes, access the Proxmox web UI to confirm successful installation.

See [[Homelab/Software/Proxmox/Quick-fixes|Quick fixes]] for post-install cleanup (enterprise repos, password, DNS).
