---
title: Growing Disk on openSUSE Leap (Proxmox VM)
description: Enlarging the root filesystem of an openSUSE Leap VM after growing its virtual disk in Proxmox.
tags:
  - openSUSE
  - Proxmox
  - Disk
  - Btrfs
  - Partition
---

# Overview

This guide covers enlarging the root filesystem of an openSUSE Leap VM after growing its virtual disk in Proxmox. It covers the common Leap layout (GPT, Btrfs root) and the pitfall where a swap partition sits after root.

> **Back up first.** Take a Proxmox backup or snapshot before touching partitions. Resizing is generally safe, but mistakes in partition editing can destroy data.

---

# 1. Grow the disk in Proxmox

**GUI:** VM → Hardware → select disk → Disk Action → Resize → enter increment (e.g., `5`).

**CLI (on the Proxmox host):**

```bash
qm config <vmid>              # find the disk name (scsi0, virtio0, sata0...)
qm resize <vmid> scsi0 +5G
```

Disks can only grow, never shrink.

---

# 2. Confirm the guest sees the new size

Inside the VM, run:

```bash
lsblk /dev/sda                # adjust device name (sda, vda, nvme0n1)
```

If the disk still shows the old size, force a rescan or reboot:

```bash
echo 1 | sudo tee /sys/class/block/sda/device/rescan
```

---

# 3. Inspect the partition layout

```bash
lsblk /dev/sda
df -hT /
```

Typical Leap layout:

| Partition | Purpose |
|-----------|---------|
| sda1 | EFI system partition (~512M) |
| sda2 | Root (Btrfs by default) |
| sda3 | Swap (often placed *after* root) |

**The key question:** is there free space directly after the root partition?

- **Root is the last partition** → go to [Path A](#path-a-root-is-the-last-partition).
- **Swap (or another partition) sits after root** → go to [Path B](#path-b-swap-sits-after-root).

If you use LVM, grow the partition, then run `pvresize` and `lvextend -r -l +100%FREE /dev/<vg>/<lv>`.

---

## Path A: Root is the last partition

```bash
sudo zypper install growpart      # if missing (may be cloud-utils-growpart)
sudo growpart /dev/sda 2          # replace 2 with the root partition number
```

Then grow the filesystem (see [step 5](#5-grow-the-filesystem)).

> **"NOCHANGE: partition cannot be grown"** means there is no free space immediately after the partition. Either the guest does not see the new size (step 2) or another partition is in the way (Path B).

---

## Path B: Swap partition after root

`growpart` and filesystem resizers can only expand into space *directly after* the partition. Swap has to be moved.

### B1. Record the old swap UUID (important)

```bash
sudo blkid /dev/sda3
grep -i swap /etc/fstab
```

Write down the UUID. It is likely referenced in more places than fstab (see [step 6](#6-fix-swap-uuid-references)).

### B2. Disable and delete swap

```bash
sudo swapoff -a
sudo parted /dev/sda rm 3
```

If parted warns that not all space is used by the GPT, choose **Fix**. This moves the backup GPT header to the new end of the disk.

### B3. Grow root to the full disk

```bash
sudo growpart /dev/sda 2
```

### B4. (Optional) Leave room for swap at the end

If you want a swap partition again, shrink root back down. **Order matters: shrink the filesystem first, then the partition.**

Example for a 15G disk wanting ~2G swap:

```bash
sudo btrfs filesystem resize 12G /          # filesystem smaller than target partition
sudo parted /dev/sda resizepart 2 13GiB     # partition END position, not size
sudo btrfs filesystem resize max /          # fill the partition exactly
```

**Notes:**

- `parted resizepart` takes the **end position** on the disk, not a size.
- Root starts around 0.5GiB, so ending at 13GiB gives a ~12.5GiB partition. The 12G filesystem fits inside it.
- Never leave a filesystem larger than its partition.
- XFS cannot be shrunk. This step only works for Btrfs and ext4. With XFS, skip the shrink and use a swapfile or a separate disk for swap.
- Skip B4 entirely if you don't need a swap partition. Remove the swap line from `/etc/fstab` and consider zram or a swapfile.

### B5. Recreate swap

```bash
sudo parted /dev/sda mkpart swap linux-swap 13GiB 100%
sudo mkswap /dev/sda3
```

**Shortcut:** reuse the old UUID to avoid step 6 entirely:

```bash
sudo mkswap -U <old-uuid> /dev/sda3
```

If you reuse the old UUID, nothing in fstab, GRUB, or the initrd needs to change.

---

# 5. Grow the filesystem

| Filesystem | Command |
|------------|---------|
| Btrfs (Leap default) | `sudo btrfs filesystem resize max /` |
| XFS | `sudo xfs_growfs /` |
| ext4 | `sudo resize2fs /dev/sda2` |

All work online. A Btrfs filesystem with many subvolumes (`/home`, `/var`, `/.snapshots`, etc.) is still one filesystem, so a single resize covers them all.

Verify:

```bash
df -h /
lsblk /dev/sda
```

---

# 6. Fix swap UUID references

Only needed if you created swap with a **new** UUID.

## Update fstab

```bash
sudo blkid /dev/sda3
sudo nano /etc/fstab        # replace the UUID on the swap line
sudo swapon -a
swapon --show
```

## Find hidden references (the boot-hang trap)

openSUSE sets `resume=` (hibernation resume device) to the swap UUID. If it still points to the deleted partition, the next boot hangs with a message like:

```
A start job is running for /dev/disk/by-uuid/<old-uuid> (1min 9s / no limit)
```

Search for the old UUID:

```bash
sudo grep -r <old-uuid> /etc /boot 2>/dev/null
```

Common locations:

- `/etc/default/grub` (`GRUB_CMDLINE_LINUX_DEFAULT` containing `resume=`)
- `/etc/kernel/cmdline`
- `/etc/dracut.conf.d/*.conf`
- `/etc/fstab`

Replace the old UUID with the new one, then regenerate the boot config:

```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo dracut -f --regenerate-all
```

> The exact bootloader tooling can differ between Leap versions. If `grub2-mkconfig` is not how your system is set up, check what the grep found and use the matching regeneration tool.

Do this **before rebooting**.

---

# Recovery: system hangs at boot after resize

Boot a live ISO (any Linux, e.g., Debian live), then:

```bash
mount /dev/sda2 /mnt
mount /dev/sda1 /mnt/boot/efi
for d in dev proc sys; do mount --rbind /$d /mnt/$d; done
chroot /mnt
```

Then either:

1. **Quick:** recreate swap with the old UUID: `mkswap -U <old-uuid> /dev/sda3`, and put the old UUID in fstab. No regeneration needed.
2. **Proper:** grep for the old UUID, replace with the new, and run the regenerate commands from step 6.

**One-time boot without resume:** at the GRUB menu press `e`, remove `resume=...` from the `linux` line, and press Ctrl+X. Then apply a permanent fix.

---

# Quick checklist

- [ ] Backup or snapshot taken
- [ ] `qm resize` done; guest `lsblk` shows new size
- [ ] Checked what sits after the root partition
- [ ] Noted the old swap UUID
- [ ] Partition grown (swap moved if necessary)
- [ ] Filesystem resized and verified with `df -h /`
- [ ] Swap recreated, with old UUID or all references updated
- [ ] fstab, GRUB/dracut references checked before reboot

---

# Lessons learned

- `NOCHANGE` from growpart means no free space directly after the partition, not that the tool failed.
- Resizing a filesystem and resizing its partition are separate operations. Shrink the filesystem first; grow the partition first.
- `parted resizepart` uses an end position, not a size.
- Swap UUIDs are referenced outside fstab (`resume=`), and a stale one causes an endless boot wait.
- Reusing the old swap UUID with `mkswap -U` avoids the whole problem.
- For future builds, put swap as the *last-resort* layout choice: a swapfile or zram avoids this problem entirely.

---

# References

- [[Homelab/Platform/Personal-VM/Opensuse/Opensuse-glossary|openSUSE Glossary]]
- [[Homelab/Software/Proxmox/Installing Proxmox on a Wi-Fi-Only Machine|Proxmox]]
