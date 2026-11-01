---
title: OpenMediaVault Storage Pool
description: Building the RAID array and BTRFS filesystem that back the NAS's shares.
tags:
  - Homelab
  - OpenMediaVault
  - Storage
---

## Overview

This page covers building the RAID array and BTRFS filesystem that back the NAS's shares, including RAID level decisions, filesystem selection, and partition planning considerations.

## Original Plan: One Filesystem Per Share

The initial plan was to give each share its own filesystem, matched to what that share needed (XFS for ISOs and AI models, BTRFS for Documents, EXT4 for PC Backups and PBS).

See [original plan screenshot](Images/omv-nas-original-per-share-filesystem-plan.png) for the original per-share filesystem plan — abandoned once `cfdisk`-based manual partitioning of `/dev/md0` turned out to be more complexity than it was worth for this NAS.

## Wiping the Drives

The four data drives had leftover, corrupted GPT headers from previous use. Wiping one of them through the OMV UI failed with an OMV-reported error, but the underlying wipe command actually succeeded.

See [GPT wipe error screenshot](Images/gpt-wipe-restores-header-from-backup-error.png) for `sgdisk` restoring a corrupted main GPT header from its backup while wiping a drive — this made the wipe command exit non-zero and OMV surface it as an error, even though the drive ended up correctly wiped.

### Root Cause

The drive's main GPT header was invalid but its backup header was intact, so `sgdisk` regenerated the main header from the backup as part of the wipe. That recovery step causes `sgdisk` to exit with code `2` instead of `0`, which OMV treats as a failure even though the wipe itself completed. When the wipe was re-run on the same drive afterward, it completed cleanly with no error.

## Enabling RAID Support

OMV's `Storage` menu has no RAID option by default — the `openmediavault-md` plugin had to be installed first (`System → Plugins`, search `md`, install, then the RAID option appears under `Storage`).

## RAID Level: 10, Then 5

RAID 10 was first created across all four drives (striped mirrors — faster writes, same usable capacity and drive-loss tolerance as RAID 5 with this drive count). After starting the initial resync, the array was deleted in favor of RAID 5 instead.

### RAID 5 Selection

Most data going onto this NAS is either re-downloadable or short-lived backups, making RAID 10's extra write speed not worth losing the ability to reconsider the layout without a second migration. The RAID 5 array was created across the same four drives and the resync (~1 hour) completed before proceeding.

## Filesystem: BTRFS on Top of mdadm, Not BTRFS RAID

OMV only allows one filesystem per physical disk, which ruled out manually partitioning the RAID device with `cfdisk` to host multiple filesystems (one per planned share). After opening `cfdisk /dev/md0` and seeing the full unpartitioned space, planning and maintaining six separate partitions (with future resizing) was decided to be more complexity than it was worth for this NAS.

See [cfdisk screenshot](Images/cfdisk-md0-manual-partition-planning-complexity.png) for the full 1.4 TiB of free space on `/dev/md0` in `cfdisk` — planning and resizing multiple manual partitions here was dropped in favor of one BTRFS filesystem with subvolumes.

### BTRFS Configuration

BTRFS was chosen for native snapshot support, but BTRFS's own multi-disk RAID is still considered experimental. BTRFS runs in **Single** profile on top of the already-redundant `mdadm` RAID 5 array, rather than letting BTRFS manage redundancy itself.

See [final layout screenshot](Images/omv-btrfs-single-profile-on-raid5-final-layout.png) for the final storage layout: a single BTRFS filesystem (profile: Single) on top of the mdadm-managed RAID 5 array `/dev/md0`.

Individual shared folders (ISOs, Documents, PC Backups, PBS, LLMs) were created as BTRFS subvolumes/paths within this one filesystem rather than as separate partitions.

## Related Documentation

- [[Homelab/Software/OpenMediaVault/Shares|Shares]] — Turning this pool into NFS shares
- [[Homelab/Software/OpenMediaVault/Openmediavault-glossary|Glossary]] — RAID/BTRFS terms
- [[Homelab/Software/OpenMediaVault/Btrfs-reference|BTRFS reference]] — How the filesystem is monitored and maintained day to day (allocation, balance, scrub, quotas)
