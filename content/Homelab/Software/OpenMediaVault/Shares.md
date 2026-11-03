---
title: OpenMediaVault Shares on Dell Vostro 260s
description: Turning the BTRFS storage pool into NFS shares, SMART monitoring, and the extra share needed for Tailscale.
tags:
  - Homelab
  - OpenMediaVault
  - Storage
---

## Overview

This page covers configuring OpenMediaVault shares, AutoFS mounting, SMART monitoring, and Tailscale-specific share configuration.

## Shared Folders and NFS

Each planned share (ISOs, Documents, PC Backups, PBS, LLMs) was created as an OMV shared folder on the BTRFS pool, given a dedicated user with owner access, and exported as an NFS share.

Clients mount shares with the standard `mount -t nfs <OMV-IP>:/<share> <mountpoint>` syntax. Each mounted share was verified writable before reliance.

## AutoFS

Client machines use AutoFS to mount these shares on demand rather than at all times.

### AutoFS Behavior

AutoFS does not show the share's root directory by default, which makes `/mnt/OMV` appear empty and looks like a broken mount — this is normal AutoFS behavior (the root only populates once a subdirectory under it is actually accessed), not a mount failure.

## SMART Monitoring

SMART monitoring was enabled per-disk under `Storage → S.M.A.R.T. → Settings`:

- **Poll interval:** 1800 seconds
- **Temperature alert:** Flags both a relative change (5°C) and an absolute maximum (45°C)

See [SMART monitoring screenshot](Images/omv-smart-monitoring-settings.png) for S.M.A.R.T. monitoring enabled with a 1800s poll interval and 45°C maximum-temperature alert.

## Extra Share for Tailscale

Each NFS export is restricted to the home lab's own subnet (`10.10.0.0/28`), so a client reaching the NAS over Tailscale instead of the LAN (e.g., the daily-driver laptop at Tailscale IP `100.68.92.125/32`) falls outside that allowed range and gets denied.

A second share was created with the same settings as the original, but scoped to the client's individual Tailscale IP instead of the LAN subnet, rather than widening the original share's allowed range.

---

See [[Homelab/Software/OpenMediaVault/Ssh-hardening|SSH hardening]] for locking down remote access to the NAS itself.
