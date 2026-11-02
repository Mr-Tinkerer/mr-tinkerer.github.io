---
title: OMV BTRFS Compression Configuration
description: Guide to enabling ZSTD compression for BTRFS filesystem on OpenMediaVault.
tags:
  - Homelab
  - OpenMediaVault
  - BTRFS
  - Compression
  - fstab
---

## Overview

This guide details enabling ZSTD compression for BTRFS filesystems on OpenMediaVault to reduce storage usage while maintaining data integrity.

---

## Step-by-Step Configuration

### 1. SSH into Machine

Access the OpenMediaVault server via SSH.

### 2. Backup Configuration

Create a backup of the current BTRFS configuration:

```bash
cp /etc/openmediavault/config.xml /etc/openmediavault/config.xml.BLK
```

### 3. Edit Configuration File

Edit the OpenMediaVault configuration:

```bash
nano /etc/openmediavault/config.xml
```

### 4. Locate BTRFS Mount Entry

Find the `mntent` (mount entry) that has `btrfs` as the file system type.

### 5. Add Compression Option

Insert `compress=zstd` into the `<opts>` field of the BTRFS mount entry.

### 6. Save Configuration

Save the file and exit the editor.

### 7. Apply Configuration Changes

Deploy the fstab changes:

```bash
sudo omv-salt deploy run fstab
```

### 8. Verify Configuration

Check if the compression setting was applied:

```bash
cat /etc/fstab | grep compress=zstd
```

Expected output should show `compress=zstd` in the fstab entry.

### 9. Reboot System

Reboot the OpenMediaVault server:

```bash
sudo reboot
```

### 10. Compress Existing Files

After reboot, compress existing files on BTRFS volumes:

```bash
btrfs filesystem defragment -r -v -czstd /export/*
```

**Note:** Without this step, only new files will be compressed. Existing files require this command to apply ZSTD compression.

---

## Verification

After completion, verify the compression is active:

```bash
btrfs filesystem usage /export
```

Look for `bytes_used` and `bytes_reserved` to confirm compression is reducing storage consumption.

---

## Troubleshooting

### Compression Not Applied

- Verify the fstab change was applied: `cat /etc/fstab | grep compress=zstd`
- Check if the system was rebooted after applying changes
- Ensure the BTRFS filesystem supports compression (all modern BTRFS installations do)

### Performance Impact

ZSTD compression adds CPU overhead during file writes. Monitor system performance if the server is CPU-bound.
