---
title: Storage pool
description: Building the RAID array and BTRFS filesystem that back the NAS's shares.
tags:
  - Homelab
  - OpenMediaVault
  - Storage
---
# Original plan: one filesystem per share

Before settling on the layout below, my initial plan was to give each share its own filesystem, matched to what that share needed (XFS for ISOs and AI models, BTRFS for Documents, EXT4 for PC Backups and PBS):

![[omv-nas-original-per-share-filesystem-plan.png]]
*The original per-share filesystem plan — abandoned once `cfdisk`-based manual partitioning of `/dev/md0` (see below) turned out to be more complexity than it was worth for this NAS.*

# Wiping the drives

The four data drives had leftover, corrupted GPT headers from previous use. Wiping one of them through the OMV UI failed with an OMV-reported error, but the underlying wipe command actually succeeded:

![[gpt-wipe-restores-header-from-backup-error.png]]
*`sgdisk` restoring a corrupted main GPT header from its backup while wiping a drive — this made the wipe command exit non-zero and OMV surface it as an error, even though the drive ended up correctly wiped.*

**Cause:** the drive's main GPT header was invalid but its backup header was intact, so `sgdisk` regenerated the main header from the backup as part of the wipe. That recovery step causes `sgdisk` to exit with code `2` instead of `0`, which OMV treats as a failure even though the wipe itself completed. When I re-ran the wipe on the same drive afterward, it completed cleanly with no error.

# Enabling RAID support

OMV's `Storage` menu has no RAID option by default — I had to install the `openmediavault-md` plugin first (`System → Plugins`, search `md`, install, then the RAID option appears under `Storage`).

# RAID level: 10, then 5

I first created a RAID 10 array across all four drives (striped mirrors — faster writes, same usable capacity and drive-loss tolerance as RAID 5 with this drive count). After starting the initial resync, I deleted the array in favor of **RAID 5** instead: most of the data going onto this NAS is either re-downloadable or short-lived backups, making RAID 10's extra write speed not worth losing the ability to reconsider the layout without a second migration. I created the RAID 5 array across the same four drives and let the resync (~1 hour) finish before proceeding.

# Filesystem: BTRFS on top of mdadm, not BTRFS RAID

OMV only allows one filesystem per physical disk, which ruled out manually partitioning the RAID device with `cfdisk` to host multiple filesystems (one per planned share) — after I actually opened `cfdisk /dev/md0` and saw the full unpartitioned space, I decided planning and maintaining six separate partitions (with future resizing) was more complexity than it was worth for this NAS:

![[cfdisk-md0-manual-partition-planning-complexity.png]]
*The full 1.4 TiB of free space on `/dev/md0` in `cfdisk` — planning and resizing multiple manual partitions here was dropped in favor of one BTRFS filesystem with subvolumes.*

I chose BTRFS for its native snapshot support, but BTRFS's own multi-disk RAID is still considered experimental — so I run BTRFS in **Single** profile on top of the already-redundant `mdadm` RAID 5 array, rather than letting BTRFS manage redundancy itself:

![[omv-btrfs-single-profile-on-raid5-final-layout.png]]
*The final storage layout: a single BTRFS filesystem (profile: Single) on top of the mdadm-managed RAID 5 array `/dev/md0`.*

I then create individual shared folders (ISOs, Documents, PC Backups, PBS, LLMs) as BTRFS subvolumes/paths within this one filesystem rather than as separate partitions.

See [[Homelab/Software/OpenMediaVault/Shares|Shares]] for turning this pool into NFS shares, [[Homelab/Software/OpenMediaVault/Openmediavault-glossary|the Glossary]] for RAID/BTRFS terms, and [[Homelab/Software/OpenMediaVault/Btrfs-reference|my BTRFS reference]] for how I actually monitor and maintain this filesystem day to day (allocation, balance, scrub, quotas).
