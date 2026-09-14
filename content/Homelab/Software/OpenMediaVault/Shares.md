---
title: Shares
description: Turning the BTRFS storage pool into NFS shares, SMART monitoring, and the extra share needed for Tailscale.
tags:
  - Homelab
  - OpenMediaVault
  - Storage
---
# Shared folders and NFS

I created each planned share (ISOs, Documents, PC Backups, PBS, LLMs) as an OMV shared folder on the BTRFS pool, gave it a dedicated user with owner access, and exported it as an [[Homelab/Software/OpenMediaVault/Openmediavault-glossary#NFS|NFS]] share. Clients mount shares with the standard `mount -t nfs <OMV-IP>:/<share> <mountpoint>` syntax; I verified each mounted share was writable before relying on it.

# AutoFS

I have client machines use [[Homelab/Software/OpenMediaVault/Openmediavault-glossary#AutoFS|AutoFS]] to mount these shares on demand rather than at all times. One point of confusion I ran into during setup: AutoFS does not show the share's root directory by default, which made `/mnt/OMV` appear empty and looked like a broken mount — this turned out to be normal AutoFS behavior (the root only populates once a subdirectory under it is actually accessed), not a mount failure.

# SMART monitoring

I enabled this per-disk under `Storage → S.M.A.R.T. → Settings`, with a poll interval of 1800 seconds and temperature reporting configured to flag both a relative change (5°C) and an absolute maximum (45°C):

![[omv-smart-monitoring-settings.png]]
*S.M.A.R.T. monitoring enabled with a 1800s poll interval and 45°C maximum-temperature alert.*

# Extra share for Tailscale

Each NFS export is restricted to the home lab's own subnet (`10.10.0.0/28`), so a client reaching the NAS over [[Homelab/Software/Tailscale/Tailscale-setup|Tailscale]] instead of the LAN — e.g. the daily-driver laptop, at its Tailscale IP `100.68.92.125/32` — falls outside that allowed range and gets denied. I fixed this by creating a second share with the same settings as the original, but scoped to the client's individual Tailscale IP instead of the LAN subnet, rather than widening the original share's allowed range.

See [[Homelab/Software/OpenMediaVault/Ssh-hardening|SSH hardening]] for locking down remote access to the NAS itself.
