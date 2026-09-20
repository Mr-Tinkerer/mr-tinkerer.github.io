---
title: libvirt external snapshots
description: How external snapshot layers map to disk states, and how I avoid snapshots that will not restore.
tags:
  - libvirt
  - qemu
  - snapshots
---

I use [[Libvirt-glossary#External snapshot|external snapshots]] on the libvirt VMs on my [[Gigabyte-gaming-a16-ga6h-overview|Gigabyte laptop]]. This page covers how the layers are laid out and what I do to keep snapshots restorable. For a snapshot that already fails to restore, see [[Libvirt-external-snapshot-recovery]].

---

## How the layers line up

When libvirt takes an external snapshot, it freezes the current disk file. It then creates a new [[Libvirt-glossary#Overlay|overlay]] named after the snapshot. The overlay receives every write made after the snapshot.

After two snapshots the chain looks like this:

```text
base.qcow2                 (original disk)
  +- vm.Snap1              (overlay created by Snap1, holds writes from Snap1 until Snap2)
       +- vm.Snap2         (overlay created by Snap2, holds writes made after Snap2)
```

The disk state at the moment of snapshot X is the [[Libvirt-glossary#Backing file and backing chain|backing file]] of the overlay named X. That is the file that was active right before X was taken.

To restore Snap2, I use `vm.Snap1`. If I layer on `vm.Snap2` instead, I get the disk as it is now, including everything written after the snapshot.

Timestamps confirm this. The frozen file's modification time matches the `-mem` file's timestamp. The newer overlay has a later one.

---

## Preventing restore failures

A live snapshot of a VM with a [[Libvirt-glossary#virtiofs|virtiofs]] device fails to restore. I pick from these options, roughly from safest to most convenient:

1. **Shut the VM down before snapshotting.** An offline snapshot has no memory file and restores cleanly.
2. **Use disk-only snapshots.** No RAM or device state gets saved, so nothing can fail on device state:
   ```bash
   virsh snapshot-create-as <vm> <snap> --disk-only --atomic
   ```
   These are [[Libvirt-glossary#Crash-consistent|crash-consistent]]. I add `--quiesce` when the [[Libvirt-glossary#QEMU Guest Agent|QEMU Guest Agent]] is installed, which makes them filesystem-consistent.
3. **Detach the virtiofs device before a live snapshot** and reattach it afterwards. I only do this when I need the RAM state.
4. **Skip overlay chains.** I use internal qcow2 snapshots, `qemu-img convert` copies of a shut-down VM, or backups from inside the guest.

---

## Keeping chains short

Every layer slows I/O slightly and adds a way for the chain to break. I do not let external snapshot chains grow long. I [[Libvirt-glossary#Flattening|flatten]] periodically, following the flatten section of [[Libvirt-external-snapshot-recovery]].

For background on the qcow2 format, see [[Virtual-disks-reference#6. Disk image formats: raw vs. qcow2]].
