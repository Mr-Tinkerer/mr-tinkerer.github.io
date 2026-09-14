---
title: OpenMediaVault Glossary
description: Terms used across the OpenMediaVault bucket.
tags:
  - glossary
---
## OpenMediaVault (OMV)

A Debian-based NAS OS with a web management UI. See [OpenMediaVault (official site)](https://www.openmediavault.org/).

## RAID

Redundant Array of Independent Disks — combining multiple physical drives into one logical device for redundancy, performance, or both. **RAID 5** stripes data with distributed parity across at least three drives, tolerating one drive failure. **RAID 10** stripes across mirrored pairs, also tolerating drive loss with faster writes than RAID 5 at the same drive count, at the same usable-capacity cost with four drives. See [RAID (Wikipedia)](https://en.wikipedia.org/wiki/RAID).

## mdadm

Linux's software RAID management tool, used by OMV's `openmediavault-md` plugin to create and manage RAID arrays such as `/dev/md0`. See [mdadm (Wikipedia)](https://en.wikipedia.org/wiki/Mdadm).

## BTRFS

A copy-on-write Linux filesystem with native snapshot support. Its own multi-disk RAID modes are considered experimental, so it is commonly layered in **Single** profile on top of a separately-redundant block device (e.g. an `mdadm` RAID array) instead. See [Btrfs (Wikipedia)](https://en.wikipedia.org/wiki/Btrfs) and [[Homelab/Software/OpenMediaVault/Btrfs-reference|my deeper BTRFS reference]] for how I actually manage it day to day (allocation model, balance, scrub, quotas, subvolumes).

## Block group

BTRFS's unit of pre-reserved disk space — before any data or metadata is written, BTRFS first carves the disk into block groups, then fills them. A block group that's been mostly emptied by deletions doesn't automatically shrink, which is the root cause behind most confusing BTRFS "out of space" errors. See [[Homelab/Software/OpenMediaVault/Btrfs-reference#How BTRFS allocates space|how BTRFS allocates space]] for the full explanation.

## Balance (BTRFS)

The BTRFS maintenance operation that rewrites and repacks sparsely-filled block groups into fewer, fuller ones, releasing the emptied ones back to unallocated space. Despite the name, it's not primarily about spreading data across multiple devices. See [[Homelab/Software/OpenMediaVault/Btrfs-reference#Balance|Balance]] for when and how I run it.

## Scrub (BTRFS)

BTRFS's background integrity checker — it reads every stored block, recomputes its checksum, and compares it to what's on record, self-healing any mismatch it can (DUP/RAID profiles) or reporting it when it can't (single-profile data). See [[Homelab/Software/OpenMediaVault/Btrfs-reference#Scrub|Scrub]].

## NFS

Network File System — a protocol for sharing directories over a network so remote clients can mount them as if they were local. This is what I use to expose OMV's shared folders (ISOs, Documents, PC Backups, PBS, LLMs) to the rest of the home lab: each share is exported as its own NFS mount, restricted to the client IPs or subnets I want to allow, so machines can read and write to the NAS without me having to copy files around by hand. See [[Homelab/Software/OpenMediaVault/Shares|Shares]] for how I set these up, and [NFS (Wikipedia)](https://en.wikipedia.org/wiki/Network_File_System).

## AutoFS

A Linux service that mounts filesystems (including NFS shares) on demand when they're accessed, and unmounts them after a period of inactivity, rather than keeping them mounted permanently. I use this on client machines instead of static `/etc/fstab` mounts so they aren't holding open connections to shares they aren't actively using — the tradeoff I ran into is that AutoFS doesn't show a share's root directory until something underneath it is actually accessed, which can make a freshly mounted share look empty or broken when it isn't. See [[Homelab/Software/OpenMediaVault/Shares#AutoFS|Shares]] and [Automounting (Wikipedia)](https://en.wikipedia.org/wiki/Automounting).
