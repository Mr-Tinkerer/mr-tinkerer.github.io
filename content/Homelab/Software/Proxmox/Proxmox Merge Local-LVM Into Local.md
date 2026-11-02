---
title: Proxmox Merge Local-LVM Into Local
description: Procedure to merge Proxmox local-lvm storage into local storage by deleting the thin pool and growing the root filesystem.
tags:
  - Homelab
  - Proxmox
  - LVM
  - Storage
  - Backup
  - WARNING
---

## Overview

This guide details the procedure to merge Proxmox local-lvm storage into local storage by deleting the local-lvm thin pool and growing the root filesystem to use all available disk space.

**WARNING: This procedure destroys every VM disk and container volume on local-lvm. Do not start until you have verified backups stored OFF the server.**

---

## Trade-Offs

- You lose LVM-thin features (fast snapshots, thin provisioning). VM disks become `.raw` or `.qcow2` files on ext4.
- Containers on directory storage cannot use snapshots.
- Backups and guest disks now share one pool. A runaway backup job can starve running VMs, so set retention limits or back up to another machine.
- `local` (root) cannot be deleted, since it holds the OS. Therefore, we grow `root` instead of the other way around.

---

## Step 0: Inspect Current Layout

Run the following commands to assess the current storage configuration:

```bash
lvs
vgs
df -h /
pvesm status
```

Note which VMs (`qm list`) and containers (`pct list`) exist, and whether `root` has any free room (`VFree` in `vgs`). If root is full, grow it first with any free VG space:

```bash
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root
```

---

## Step 1: Copy Existing Backups to External Drive

Do this first so the existing backup history survives the wipe. Backups on `local` normally live in `/var/lib/vz/dump`.

### Mount External Drive

Mount the external drive (Linux filesystem such as ext4 or xfs; avoid NTFS/exFAT) and confirm it is a real mount:

```bash
findmnt /mnt/ext
df -h /mnt/ext
```

### Check Backup Size

Check the size of what you are copying and the space available:

```bash
du -sh /var/lib/vz/dump
df -h /mnt/ext
```

### Copy Backups

Copy everything, preserving attributes:

```bash
mkdir -p /mnt/ext/PVE_BACKUPS_OLD
rsync -avh --progress /var/lib/vz/dump/ /mnt/ext/PVE_BACKUPS_OLD/
```

### Verify Copy

Verify the copy (re-running should transfer nothing; add `--checksum` for a stricter check):

```bash
rsync -avhc --dry-run /var/lib/vz/dump/ /mnt/ext/PVE_BACKUPS_OLD/
```

### Optional: Copy Templates and ISOs

Optionally also copy ISOs and templates you want to keep:

```bash
rsync -avh /var/lib/vz/template/ /mnt/ext/PVE_TEMPLATES/
```

---

## Step 2: Create Fresh Backups of Every Guest

List your guest IDs (`qm list`, `pct list`), then back them all up to the external drive:

```bash
vzdump <ID1> <ID2> <ID3> --dumpdir /mnt/ext/PVE_BACKUPS \
  --mode stop --compress zstd --notes-template "before lvm wipe"
```

Options:
- `--mode stop` shuts each guest down for a fully consistent backup, then restarts it.
- `--notes-template` replaces the default `{{guestname}}` note with your text.
- Use `--all` instead of listing IDs to back up everything.

### Verify Each Archive

```bash
ls -lh /mnt/ext/PVE_BACKUPS/
zstd -t /mnt/ext/PVE_BACKUPS/vzdump-*-<ID>-*.zst
tail -n 3 /mnt/ext/PVE_BACKUPS/vzdump-*-<ID>-*.log
# Expect: "Backup job finished successfully"
```

---

## Step 3: Confirm Backups from Separate Machine

Plug the external drive into (or share it with) another computer and confirm that:

- Every guest has a backup file with a recent timestamp.
- File sizes look reasonable (not 0 bytes or a few KB).
- The files can be read and copied.

**Do not proceed until this is done. After the wipe, this drive may be your only copy.**

---

## Step 4: Stop Guests and Disable Automation

Stop all VMs and containers:

```bash
for id in <IDs>; do qm shutdown $id --timeout 60 || qm stop $id; done
qm list
# All should show "stopped"
pct shutdown <id>
# For containers
```

Also disable scheduled backup jobs (Datacenter, Backup) and VM/container "Start at boot" so nothing touches disks mid-wipe.

---

## Step 5: Remove Local-LVM Storage Entry

Remove the local-lvm storage via CLI:

```bash
pvesm remove local-lvm
```

Or via GUI: Datacenter → Storage → `local-lvm` → Remove.

---

## Step 6: Delete Thin Pool

Delete the local-lvm thin pool:

```bash
lvremove /dev/pve/data
```

Answer `y` to the prompts (this removes the pool and every guest disk in it). Confirm only `root` and `swap` remain:

```bash
lvs
```

---

## Step 7: Grow Root Into Freed Space

Expand the root filesystem to use all available space:

```bash
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root
df -h /
```

`resize2fs` grows ext4 online; no reboot is needed. `df -h /` should now show nearly the whole disk.

---

## Step 8: Enable All Content Types on Local

Enable all content types on the local storage.

### Via GUI

Datacenter → Storage → `local` → Edit → Content:
- [x] Disk image
- [x] Container
- [x] Backup
- [x] ISO image
- [x] Container template
- [x] Snippets

### Via CLI

```bash
pvesm set local --content backup,iso,vztmpl,snippets,images,rootdir
pvesm status
# local should be active with ~full disk free
```

---

## Step 9: Clear Stale Guest Configs and Temp Files

The old guest configs still point at `local-lvm`, which no longer exists. If you try to restore over them (even with `--force`), the restore can fail with:

```
storage 'local-lvm' does not exist
```

Since the backups contain each guest's config, it is safe to delete the stale ones and restore onto a clean slate:

```bash
# VMs
rm -f /etc/pve/qemu-server/<ID>.conf
# Containers
rm -f /etc/pve/lxc/<ID>.conf
# Leftovers from any failed restore attempts
rm -rf /var/tmp/vzdumptmp*

ls /etc/pve/qemu-server/ /etc/pve/lxc/
# Should be empty of the guests you're restoring
```

---

## Step 10: Restore Guests onto Local

Use the fresh "before lvm wipe" backups (newest timestamp for each ID). **Do not use `--force`; the configs are already gone.**

### Restore VMs

```bash
cd /mnt/ext/PVE_BACKUPS

# VMs
qmrestore vzdump-qemu-<ID>-<timestamp>.vma.zst <ID> --storage local
```

### Restore Containers

```bash
# Containers
pct restore <ID> vzdump-lxc-<ID>-<timestamp>.tar.zst --storage local
```

### Check Results

```bash
qm list
qm config <ID> | grep -E 'scsi0|virtio0|sata0|efidisk0'
# Should show local:<ID>/vm-<ID>-disk-N.raw
```

Start each guest and confirm it boots and works.

---

## Step 11: Move Old Backups Back

Once every guest is restored and verified, return the original backup history to where Proxmox expects it:

```bash
rsync -avh --progress /mnt/ext/PVE_BACKUPS_OLD/ /var/lib/vz/dump/
```

Before deleting anything from the external drive, confirm the transfer and check free space:

```bash
rsync -avhc --dry-run /mnt/ext/PVE_BACKUPS_OLD/ /var/lib/vz/dump/
# Should transfer nothing
df -h /
```

Restore templates/ISOs the same way if you copied them (`/var/lib/vz/template/`). Proxmox picks the files up automatically; check them under Datacenter → Storage → `local` → Backups.

Keep the external copies until you are confident everything is healthy (a week or more is sensible). The fresh "before lvm wipe" backups can be kept or deleted at your discretion.

---

## Step 12: Re-Enable Automation and Set Retention

- Re-enable scheduled backup jobs and "Start at boot" on guests.
- Set a retention policy (Datacenter → Backup → job → Retention), for example keep-last 3 and keep-weekly 4, so backups cannot fill the pool.
- Strongly consider pointing scheduled backups at a different disk, NAS, or Proxmox Backup Server, since a single pool means a full disk affects everything.

---

## Troubleshooting

| Symptom | Cause / Fix |
|---------|-------------|
| `storage 'local-lvm' does not exist` on restore | Old config still references it. Do Step 9, restore without `--force`, and pass `--storage local`. |
| Restore says storage `local` doesn't allow images | Content types not enabled. Do Step 8. |
| `findmnt` prints nothing for the drive | It is not mounted; you would be writing to root. Mount it first. |
| Root at 100% before starting | Grow root with free VG space first (Step 0), or delete old backups after copying them off. |
| Guest has a lock | `qm unlock <ID>` (or `pct unlock <ID>`). |
| `vzdump` can't write to the drive | Check filesystem type and permissions; use ext4/xfs, not NTFS/exFAT. |

---

## Quick Reference

```bash
# Backup
vzdump <IDs> --dumpdir /mnt/ext/PVE_BACKUPS --mode stop --compress zstd --notes-template "before lvm wipe"

# Wipe and merge
pvesm remove local-lvm
lvremove /dev/pve/data
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root

# Enable content on local
pvesm set local --content backup,iso,vztmpl,snippets,images,rootdir

# Restore
rm -f /etc/pve/qemu-server/<ID>.conf
qmrestore <archive>.vma.zst <ID> --storage local
```

---

## Entry Q&A

```json
{
  "questions": [
    {
      "id": "q001",
      "type": "missing-url",
      "question": "What is the GitHub URL for any custom scripts or automation used in this procedure?",
      "context": "The draft mentions scripts but does not provide GitHub URLs for them.",
      "related": ["Homelab/Software/Proxmox/Proxmox-installation.md"],
      "answer": null
    }
  ]
}
```
