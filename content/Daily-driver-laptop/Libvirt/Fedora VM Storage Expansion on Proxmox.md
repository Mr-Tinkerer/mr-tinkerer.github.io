---
title: Fedora VM Storage Expansion on Proxmox
description: Guide to expanding storage for a Fedora Linux VM running on Proxmox with LVM and XFS filesystem.
tags:
  - Homelab
  - Proxmox
  - Fedora
  - LVM
  - XFS
  - Virtualization
---

## Overview

This guide details the process of increasing the virtual disk size of a Fedora Linux VM running on Proxmox, assuming the typical LVM plus XFS configuration. The procedure involves resizing the virtual disk in Proxmox, then expanding the partition, physical volume, logical volume, and XFS filesystem inside the guest.

---

## Prerequisites

- **Backup the VM** or take a Proxmox snapshot before resizing.
- Access to the Proxmox web interface.
- SSH access to the Fedora VM.
- The `cloud-utils-growpart` package installed in the guest (install with `sudo dnf install cloud-utils-growpart` if missing).

---

## Step 1: Shut Down the VM

Shut down the Fedora VM before resizing its virtual disk:

```bash
sudo shutdown -h now
```

Alternatively, shut down the VM through the Proxmox web interface.

---

## Step 2: Increase Virtual Disk Size in Proxmox

In the Proxmox web interface:

1. Select the Fedora VM.
2. Open Hardware.
3. Select the virtual disk.
4. Choose Disk Action → Resize.
5. Enter the additional storage to add (not the final disk size).
6. Click Resize disk.
7. Start the VM.

Example: If the current disk is 32 GB and the desired size is 64 GB, increase it by 32 GB.

---

## Step 3: Verify New Disk Size

After the VM starts, run:

```bash
lsblk
```

The disk size should reflect the new capacity. However, the LVM partition will still show the old size, indicating that the newly available space must be added to the partition.

---

## Step 4: Identify LVM Partition

Run the following commands to identify the storage layout:

```bash
lsblk -f
sudo pvs
```

Example output from `pvs`:

| PV         | VG     | Fmt  | Attr | PSize   | PFree |
|------------|--------|------|------|---------|-------|
| /dev/sda3  | fedora | lvm2 | a--  | <30.00g | 0     |

In this example:

- **Disk:** `/dev/sda`
- **LVM partition:** `/dev/sda3`
- **Partition number:** `3`
- **Volume group:** `fedora`

Common device names include `/dev/sda3`, `/dev/vda3`, and `/dev/nvme0n1p3`. Always identify your actual devices before proceeding.

---

## Step 5: Expand the LVM Partition

Use `growpart` to extend the partition into the newly available disk space:

```bash
sudo growpart /dev/sda 3
```

Replace `/dev/sda` and `3` with your actual disk and partition number.

Verify the partition size:

```bash
lsblk
```

The LVM partition should now occupy most or all of the newly available disk space.

---

## Step 6: Expand the LVM Physical Volume

Tell LVM that the physical volume has become larger:

```bash
sudo pvresize /dev/sda3
```

Verify the result:

```bash
sudo pvs
```

The `PSize` column should have increased, and `PFree` should now be greater than zero.

---

## Step 7: Identify Root Logical Volume

List the logical volumes:

```bash
sudo lvs
```

Example output:

| LV    | VG     | Attr       | LSize |
|-------|--------|------------|-------|
| root  | fedora | -wi-ao---- | 28.00g|
| swap  | fedora | -wi-ao---- |  2.00g|

The root logical volume is commonly named `root`. You can also inspect the filesystem mounted at `/`:

```bash
findmnt /
```

This confirms the root filesystem type and identifies the device backing `/`.

---

## Step 8: Expand the Root Logical Volume

Allocate all remaining free space in the volume group to the root logical volume:

```bash
sudo lvextend -l +100%FREE /dev/fedora/root
```

Replace `fedora` and `root` with your actual volume group and logical volume names.

Verify the new size:

```bash
sudo lvs
```

The root logical volume should now be larger.

---

## Step 9: Expand the XFS Filesystem

The logical volume is now larger, but the XFS filesystem must also be expanded:

```bash
sudo xfs_growfs /
```

**Note:** Do not use `resize2fs` for XFS. That utility is intended for ext2/ext3/ext4 filesystems. Use `xfs_growfs` for a mounted XFS filesystem.

---

## Step 10: Verify Results

Check the available space on the root filesystem:

```bash
df -h /
```

Example output:

| Filesystem               | Size  | Used | Avail | Use% | Mounted on |
|--------------------------|-------|------|-------|------|------------|
| /dev/mapper/fedora-root  | 60G   | 15G  | 45G   | 25%  | /          |

Check the block devices:

```bash
lsblk
```

Check the LVM configuration:

```bash
sudo pvs
sudo vgs
sudo lvs
```

A successful configuration shows the entire chain expanded from Proxmox virtual disk through partition, physical volume, logical volume, to the XFS filesystem mounted at `/`.

---

## Quick Reference

Assuming the following configuration:

| Setting          | Value            |
|------------------|------------------|
| Disk             | `/dev/sda`       |
| LVM partition    | `/dev/sda3`      |
| Volume group     | `fedora`         |
| Root logical volume | `root`         |
| Root filesystem  | XFS              |

Run the following commands in order, substituting your actual device and volume names:

```bash
# Inspect the current layout
lsblk -f
sudo pvs
sudo vgs
sudo lvs
findmnt /

# Expand the partition
sudo growpart /dev/sda 3

# Expand the LVM physical volume
sudo pvresize /dev/sda3

# Allocate all free LVM space to the root logical volume
sudo lvextend -l +100%FREE /dev/fedora/root

# Expand the XFS filesystem
sudo xfs_growfs /

# Verify the results
df -h /
lsblk
sudo pvs
sudo vgs
sudo lvs
```

---

## Troubleshooting

### growpart: command not found

Install the required package:

```bash
sudo dnf install cloud-utils-growpart
```

Then retry with the correct disk and partition number.

### pvresize reports no change

Check the partition size:

```bash
lsblk
```

Then check the physical volume:

```bash
sudo pvs
```

If the partition was not expanded, run `growpart` again with the correct disk and partition number.

### lvextend reports that there is no free space

Check the volume group:

```bash
sudo vgs
```

Look at the `VFree` column. If it shows `0`, the volume group has no unallocated space available.

If the physical volume has not been expanded, run:

```bash
sudo pvresize /dev/sda3
```

Replace `/dev/sda3` with your actual LVM partition.

### df -h still shows the old size

First, verify that the logical volume has been expanded:

```bash
sudo lvs
```

If the logical volume is larger but the filesystem still reports its previous size, run:

```bash
sudo xfs_growfs /
```

Then verify again:

```bash
df -h /
```

---

## Recommended Pre-Resize Checklist

When working with an unfamiliar Fedora VM, collect the storage layout before making changes:

```bash
lsblk -f
sudo pvs
sudo vgs
sudo lvs
findmnt /
```

Document the following:

- **Disk:** For example, `/dev/sda`
- **LVM partition:** For example, `/dev/sda3`
- **Volume group:** For example, `fedora`
- **Root logical volume:** For example, `fedora/root`
- **Filesystem:** For example, XFS
- **Mount point:** `/`

Once identified, the resize workflow is:

```
Increase virtual disk in Proxmox
              ↓
           growpart
              ↓
           pvresize
              ↓
           lvextend
              ↓
         xfs_growfs /
              ↓
           Verify
           df -h /
```

---

## Summary

For a Fedora VM using LVM and XFS, increasing the virtual disk size in Proxmox is only the first step. The complete process is:

1. **Proxmox:** Increase the virtual disk size.
2. **Partition:** Expand the LVM partition with `growpart`.
3. **LVM physical volume:** Expand it with `pvresize`.
4. **LVM logical volume:** Allocate the additional space with `lvextend`.
5. **XFS filesystem:** Expand it with `xfs_growfs`.
6. **Verification:** Confirm the resulting sizes with `df -h`, `lsblk`, `pvs`, `vgs`, and `lvs`.

**The most important rule is to identify your actual disk, partition, volume group, and logical volume before running any resize commands.**
