---
title: OpenMediaVault Filesystem Migration from BTRFS to EXT4
description: Migrating the OpenMediaVault NAS storage from BTRFS to EXT4 due to filesystem confusion and deletion issues.
tags:
  - OpenMediaVault
  - Migration
  - Filesystem
  - BTRFS
  - EXT4
---

# Overview

BTRFS caused confusion on filesystem usage, particularly with file deletion. The decision was to migrate from BTRFS to EXT4 by copying data to an external drive, wiping the array, recreating it on EXT4, and restoring the data.

> **Total space reduction:** 1.3TB → 890GB

---

# Prerequisites and Preparation

## Hardware

- External drive with sufficient space for the data being migrated
- Access to SSH the OpenMediaVault (OMV) server

## Pre-migration steps

1. Grab an external drive
2. Double check there's no important data on it
3. Format the drive to be EXT4
4. Turn off the Proxmox (PVE) box
5. Turn off autofs on the laptop

---

# Migration Steps

## 1. Prepare the external drive

Plug the external drive into the NAS and SSH into OMV.

Check what drive it is via:

```bash
lsblk
```

Mount it:

```bash
mount /dev/sdX1 /mnt
```
(Replace `sdX1` with the actual device)

## 2. Copy the shares

Navigate to `/export` and copy the shares that are needed:

```bash
rsync -avzlHP --progress /export/SHARE /mnt/
```

> **Note:** If there's not enough space, copy only the shares you need. For this migration, everything except PC backups was copied.

## 3. Flush and unmount

Run `sync` to flush all write caches:

```bash
sync
```

Unmount the drive:

```bash
umount /mnt
```

## 4. Prepare the NAS

Now prepare the NAS for the filesystem migration:

1. Delete the NFS shares
2. Delete the shared folders
3. Delete the RAID 5 array

## 5. Recreate the storage

1. Recreate the RAID 5 array
2. Partition EXT4 on the array
3. Create the following shares:
   - `Arrrrgh`
   - `ISOs`
   - `LLMs`
   - `Other_Backups`
   - `PBS`
   - `PC_Backups`

### Set permissions

File system permissions are set as follows:

- `Arrrrgh`, `ISOs`, and `LLMs`: owner: `rwx`, group: `rwx`, other: `rx`
- `Other_Backups`, `PBS`, and `PC_Backups`: owner: `rwx`, group: `rwx`, other: `none`

## 6. Configure NFS shares

Create the NFS shares for all file systems with the following settings:

- All set to read-write permission
- All set to listen on `0.0.0.0/1` (`0.0.0.0/0` broke the mount)
- `ISOs` and `PBS` have the `no_root_squash` flag appended

## 7. Restore the data

Plug the external drive into the system, SSH into OMV, mount the drive, and copy the files to the correct spot:

```bash
rsync -avzlHP --progress /mnt/SHARE/* /export/SHARE/
```

## 8. Update configuration

Modify the `omv.nfs` file to point to the new shares.

---

# Post-migration

## 1. Enable autofs

Turn on autofs.

## 2. Clean up backups

Delete the auto documents backup service from the laptop, as it's now pointless without the BTRFS snapshots. The Borg backups can still be used for documents.

## 3. Restore PC backups

Rerun the PC Borg backup, as it was the one share that wasn't backed up.

## 4. Results

The migration resulted in a space reduction from 1.3TB to 890GB.

## 5. Reboot

Turn on the Proxmox (PVE) box.

---

# References

- [[Homelab/Software/OpenMediaVault/OpenMediaVault-Glossary|OpenMediaVault Glossary]]
- [[Homelab/Software/OpenMediaVault/Storage-pool|Storage Pool]]
