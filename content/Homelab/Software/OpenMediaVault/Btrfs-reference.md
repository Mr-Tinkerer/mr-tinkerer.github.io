---
title: BTRFS reference
description: A deeper practical reference for the BTRFS filesystem I run on top of the NAS's mdadm RAID array — how it allocates space, how to read its usage output, and how I keep it healthy.
tags:
  - Homelab
  - OpenMediaVault
  - Storage
---
# Why this page exists

[[Homelab/Software/OpenMediaVault/Storage-pool|Storage pool]] covers the decision to run BTRFS in Single profile on top of the [[Homelab/Software/OpenMediaVault/Openmediavault-glossary#mdadm|mdadm]]-managed RAID 5 array, and [[Homelab/Software/OpenMediaVault/Openmediavault-glossary#BTRFS|the Glossary]] covers the term at a high level. This page goes deeper into how BTRFS actually manages space day to day, since understanding that is what lets me tell a genuinely full NAS apart from one that just needs a balance.

# How BTRFS allocates space

BTRFS never writes directly to raw disk. It first carves the disk into **block groups** — pre-reserved containers of space — and only then fills those containers with data or metadata. This two-layer model is why `df` and BTRFS's own tools can disagree, and why "plenty of free space" can still produce an out-of-space error:

| Term | Meaning |
|---|---|
| **Device size** | Total physical size of all drives in the filesystem |
| **Allocated** | How much of the disk has been carved into block groups |
| **Used** | How much of the allocated space actually contains data |
| **Unallocated** | Raw disk space not yet assigned to any block group |

Block groups that have been mostly emptied (after deleting files or old snapshots) don't automatically shrink — a block group that's only 10% full stays fully allocated. Over time this can leave `Unallocated` very small even while `df` still reports free space, and specifically starves BTRFS of room to create new **metadata** block groups (data and metadata block groups aren't interchangeable — BTRFS can't repurpose a mostly-empty data block group for metadata on the fly). That's the single most confusing BTRFS failure mode, and the fix is always [[Homelab/Software/OpenMediaVault/Btrfs-reference#Balance|balance]].

## Reading `btrfs filesystem usage`

```bash
sudo btrfs filesystem usage /srv/dev-disk-by-uuid-.../
```

The overall section reports `Device size`, `Device allocated`, `Device unallocated`, `Used`, and two free-space figures: **Free (estimated)** assumes metadata grows proportionally to current usage, while **Free (minimum)** is the worst case — metadata expanding to fill every metadata block group before any more can be allocated. The gap between them exists because metadata uses the **DUP** profile (two copies, even on a single device), so each new metadata block group costs double the raw space. I watch **Free (minimum)** specifically and don't let it drop below roughly 20 GiB, since that's the number that actually predicts an out-of-space error.

The block-group section below that lists each block-group type (`Data`, `Metadata`, `System`) with its profile, allocated size, and used size — a `Metadata,DUP: Size:3.00GiB` entry showing `6.00GiB` on the underlying device is expected, not a bug: DUP writes every byte twice.

## Profiles: why metadata is mirrored but data isn't

| Profile | Copies | Used for |
|---|---|---|
| **single** | 1 | Data, on a single logical device (this NAS's BTRFS filesystem sits on one `mdadm` device, `/dev/md0`) |
| **DUP** | 2 (same device) | Metadata and System, by default — protects the filesystem's own structure |
| **RAID1** | 2 (different devices) | Not used here — redundancy is already handled one layer down by `mdadm`, so BTRFS itself runs Single/DUP rather than duplicating that work |

Metadata gets mirrored even on a single-device BTRFS filesystem because losing it is categorically worse than losing data: losing a data block corrupts one file, but losing metadata can make the entire filesystem unable to locate or verify *any* file. Mirroring metadata costs only a percent or two of total disk space, whereas mirroring all data would halve usable capacity — which is exactly why I rely on the RAID 5 array underneath for data redundancy instead of asking BTRFS to do it twice.

# Balance

Balance's name undersells what it does — it isn't primarily about spreading data across devices, it's about **rewriting and repacking block groups** so mostly-empty ones can be released back to `Unallocated`. I run it whenever `Device allocated` looks much larger than `Used`, whenever `Unallocated` drops below ~20 GiB, or after a large deletion (a big file removed, or several snapshots cleaned up at once):

```bash
# Routine — only repacks block groups less than 70% full
sudo btrfs balance start -dusage=70 -musage=70 /srv/dev-disk-by-uuid-.../

# Check progress
sudo btrfs balance status /srv/dev-disk-by-uuid-.../
```

There's no fixed schedule for this NAS — I balance reactively (after a big cleanup, or if `Unallocated` looks low), not on a timer, since unnecessary balancing just adds write amplification for no benefit.

# Scrub

Scrub is BTRFS's background integrity checker — it reads every data and metadata block, recomputes its checksum, and compares it against the stored one. On a DUP/RAID profile a mismatch self-heals from the good copy automatically; on a `single`-profile block (this NAS's data blocks), a mismatch is detected and reported but can't be auto-repaired — I'd need to restore that file from elsewhere. This is one of BTRFS's real advantages over a plain `mdadm` + ext4 stack: `mdadm` alone has no idea if the bytes it's mirroring are already silently corrupted (bit rot, a failing sector, an incomplete write from a power blip); BTRFS actually knows what every block *should* checksum to.

```bash
sudo btrfs scrub start /srv/dev-disk-by-uuid-.../
sudo btrfs scrub status /srv/dev-disk-by-uuid-.../
```

I run this monthly (OMV can schedule it, or a systemd timer works just as well), since it's read-only in the normal case and just uses background I/O.

# Quotas (qgroups)

BTRFS can track and cap per-subvolume space usage via qgroups, but every quota-tracked write pays an accounting-update cost — on a filesystem with several subvolumes this can be a real (20–50%) write-performance hit, and qgroup accounting has a known history of drifting out of sync and needing a slow full rescan. For a single-NAS use case like this one, where I'm not selling storage to separate tenants, `compsize`/`btrfs filesystem du` on demand gives me the same visibility without that ongoing cost, so I leave quotas disabled:

```bash
sudo btrfs qgroup show /srv/dev-disk-by-uuid-.../
sudo btrfs quota disable /srv/dev-disk-by-uuid-.../
```

# Subvolumes are how I split this pool into shares

A subvolume is an independently addressable filesystem tree inside one BTRFS filesystem — it looks like a normal directory but can be mounted, snapshotted, and managed on its own. This is the mechanism [[Homelab/Software/OpenMediaVault/Storage-pool|Storage pool]] relies on to split the single BTRFS filesystem on `/dev/md0` into separate shares (ISOs, Documents, PC Backups, PBS, LLMs) instead of separate partitions. A snapshot is just a subvolume created with another subvolume's contents as its starting point — because of copy-on-write, taking one is nearly instant and free until the live data and the snapshot start to diverge.

```bash
sudo btrfs subvolume list /srv/dev-disk-by-uuid-.../
```

# Measuring actual usage — `du` lies on BTRFS

Regular `du` is unreliable here for two reasons: it double-counts extents shared between subvolumes/snapshots (counting the same physical bytes once per reference instead of once), and it reports logical, uncompressed size rather than actual bytes on disk. For this NAS I reach for:

| Tool | What it actually tells me |
|---|---|
| `sudo btrfs filesystem usage <path>` | Overall filesystem health / free space |
| `sudo compsize -x <path>` | Physical, compression-aware bytes actually on disk for a subvolume |
| `sudo btrfs filesystem du -s <path>` | The **Exclusive** column — how much space deleting this directory would actually free, accounting for extent sharing with snapshots |

# Large monolithic files: PBS/backup images and `nocow`

Anything on this NAS that behaves like a single large file under constant random writes (a Proxmox Backup Server datastore image, a VM disk if one ever lands on a BTRFS-backed share) is the worst case for BTRFS's copy-on-write model — every small write relocates data to a new extent, scattering the file across the disk over time. The standard fix is disabling copy-on-write for that specific file or its containing subvolume with `chattr +C`, which makes BTRFS overwrite those files in place like a traditional filesystem instead:

```bash
sudo btrfs subvolume create /srv/dev-disk-by-uuid-.../backups
sudo chattr +C /srv/dev-disk-by-uuid-.../backups
```

The tradeoff: nocow files lose BTRFS checksumming (no silent-corruption detection) and can't be included in snapshots — acceptable for something like PBS, which already does its own integrity verification. `+C` only affects *new* writes, so an existing directory of files has to be moved out, recreated as a nocow subvolume, and copied back in with `cp -a` (never `mv` within the same filesystem, since that can silently preserve the old CoW extents instead of writing fresh nocow ones) — I haven't needed to do this on this NAS yet, but it's the playbook if the PBS share ever needs it. I don't currently have anything on this NAS that fits this case badly enough to be worth the tradeoff, so all shares here stay CoW for now.

One important side effect: nocow doesn't prevent block-group exhaustion — it can actually make it worse, since a large nocow file occupies a concentrated run of block groups at high density, and deleting it hollows those specific block groups out all at once rather than spreading the loss thinly across many groups. Any time I delete or shrink a large file like this, a [[Homelab/Software/OpenMediaVault/Btrfs-reference#Balance|balance]] afterward is worth doing, not optional.

# Routine maintenance I actually do on this NAS

- **Monthly:** `btrfs scrub start`, then check `btrfs scrub status` once it finishes.
- **After a large deletion or snapshot cleanup:** `btrfs balance start -dusage=70 -musage=70`.
- **Whenever something feels off:** `btrfs filesystem usage`, plus `dmesg | grep -i btrfs` for any reported errors.

A filesystem in good shape has `Used` close to `Allocated` (no wasted empty block groups), `Unallocated` comfortably above ~20 GiB, and no BTRFS errors in `dmesg`. If `Allocated` is much bigger than `Used` and `Unallocated` is thin, that's the balance signal above, not a sign anything is actually broken.
