---
title: OpenMediaVault Installation on Dell Vostro 260s
description: Wiping old partitions and installing OpenMediaVault onto the Dell Vostro 260s NAS hardware.
tags:
  - Homelab
  - OpenMediaVault
---

## Overview

This guide covers OpenMediaVault installation, including partition cleanup and initial configuration.

## Pre-Installation Preparation

### Partition Cleanup

Before installation, wipe leftover partitions from previous operating systems:

1. Boot a live Arch Linux environment
2. List partitions with `lsblk`
3. Use `fdisk` to write a fresh GPT partition table over each drive

## Installation

### Boot Media

Boot the installer from Ventoy USB stick.

### Post-Installation Verification

After installation completes, access the OpenMediaVault web UI to confirm successful installation.

## Network Configuration

### Interface Configuration

Under `network → Interfaces`, switch the interface from DHCP to a static address matching the [[Homelab/Logical/Home-network/Addressing-plan|addressing plan]].

### DNS Configuration

Point the DNS server at the router, which also runs [[Homelab/Software/OpenWrt/Dhcp-and-dns-configuration|the home lab's custom DNS]].

See [[Homelab/Software/OpenMediaVault/Quick-fixes|Quick fixes]] for post-install cleanup.
