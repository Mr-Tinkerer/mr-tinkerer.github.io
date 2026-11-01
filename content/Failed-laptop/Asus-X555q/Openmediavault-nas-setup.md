---
title: OpenMediaVault NAS Setup on Asus X555q
description: Installing OpenMediaVault as a Proxmox VM, selecting BTRFS filesystem, and configuring NFS shares with ACL ownership fixes.
tags:
  - Asus-X555q
  - OpenMediaVault
  - Networking
---

## Overview

This guide covers OpenMediaVault installation as a Proxmox VM on the Asus X555q laptop, including filesystem selection, network configuration, and NFS share troubleshooting.

## Why OpenMediaVault

### Hardware Constraints

TrueNAS was ruled out due to minimum requirements exceeding available resources:

- 2 cores minimum
- 8 GB RAM minimum
- 16 GB boot device
- Two identically-sized storage devices

See [TrueNAS requirements](Images/truenas-scale-minimum-hardware-requirements.png) for the constraint that ruled it out.

### Selection

OpenMediaVault (OMV) was chosen as a lighter Debian-based NAS OS, avoiding manual functionality assembly from a bare Linux distro.

## VM Layout

### Configuration

- **CPU:** 2 cores
- **RAM:** 4 GB
- **Network:** Ethernet NIC connected to switch
- **Disks:** Two virtual disks
  - 10 GB — OMV OS disk
  - 500 GB — NAS storage disk

Most NAS software expects the OS on a separate disk from managed storage shares.

See [VM layout diagram](Images/omv-vm-two-disk-layout-diagram.png) for the two-disk configuration.

## Installation and Network Configuration

### Language Configuration

The installer language was initially set to Russian. The installation completed successfully, and a repeat installation used the correct language for day-to-day use. Full reinstall from ISO takes only a few minutes.

### Installation Failure

The install crashed the underlying Proxmox host with a kernel panic partway through. A manual reboot recovered Proxmox, requiring only OMV VM reinstallation.

### Network Configuration

Static IP was configured (no gateway or DNS) following the private switch design rationale.

### Web UI Access Issue

After installation, the web UI was unreachable over both HTTP and HTTPS despite the VM responding to ping.

### Root Cause

The browser (LibreWolf and Chromium tested) silently rewrote `http://` requests to `https://` without a visible redirect, while OMV's UI was not listening on HTTPS.

See [LibreWolf screenshot](Images/librewolf-http-silently-upgraded-to-https-connection-refused.png) for the HTTPS upgrade behavior.

### Resolution

Forcing the browser to keep the `http://` scheme by re-entering the URL resolved access. Default `admin` / `openmediavault` credentials worked.

## Filesystem Selection

### Supported Filesystems

OMV does not support ZFS. Available options: BTRFS, EXT4, F2FS, JFS, XFS.

### Selection Rationale

BTRFS was chosen for:

- Built-in snapshots
- Self-healing filesystem
- Compression

These features are deliberately omitted from EXT4 in favor of simplicity.

### Configuration

The 500 GB disk was formatted BTRFS in single-device mode and mounted without issue.

## NFS Share Configuration

### Shared Folder Setup

- **Path:** `NAS` at the root of the BTRFS filesystem
- **Permissions:** Administrator: read/write, Users: read/write, Others: no access
- **Protocol:** NFS (preferred over SMB/CIFS for Linux clients)

The client device was configured with the `nfs-utils` package and the NFS client service enabled.

### NFS Service Configuration

**NFS is disabled by default in OMV** even after share configuration. The service must be enabled under `Services → NFS → Settings`.

See [NFS disabled screenshot](Images/omv-nfs-service-disabled-by-default.png) for the service settings.

### Mount Verification

With NFS enabled, the share mounted successfully from the client:

```bash
mount -t nfs <omv-ip>:/ /mnt/OMV
```

See [mounted share screenshot](Images/omv-nfs-share-mounted-with-restrictive-permission-bits.png) for the mounted export with `drwxrws---` permission bits owned by root.

## Permission Denied Issue

### Problem

Even after granting the client's user account read/write access, and even when `su`-ing to root on the client, access was denied:

See [permission denied screenshot](Images/nfs-share-permission-denied-even-as-root.png) for the failed `cd` attempt as both normal user and root.

### Root Cause

The shared folder's ACL had the owning user set to `root`, blocking access regardless of permission bits. OMV's regular "Permissions" tab sets classic owner/group/other mode bits but does not modify the ACL.

See [ACL screenshot](Images/omv-shared-folder-acl-owner-set-to-root.png) for the ACL showing root as the owning user.

### Resolution

Switching the ACL's owning user to the client's user account resolved the issue. The folder became writable and file creation succeeded:

See [resolution screenshot](Images/nfs-share-writable-after-acl-owner-fix.png) for confirmation.

### Final Configuration

After the fix, the NFS mount was reconfigured to point directly at the share's export path (`<omv-ip>:/export/NAS`) rather than OMV's parent `/export` pseudo-filesystem root, avoiding inherited permission surprises.

## Gotchas

1. **Service Not Enabled** — If an OMV share is unreachable after configuration, check whether the NFS/SMB service itself is enabled. Configuring a share does not enable the underlying service.
2. **ACL Ownership** — If a share is unwritable even for root, check the shared folder's ACL owner in OMV, not just Unix permission bits. OMV's basic permission editor and ACL editor are separate; only the ACL editor changes file ownership.
3. **Export Path** — Prefer mounting a share's specific export path over OMV's parent `/export` pseudo-filesystem root to avoid inherited permission issues.
