---
title: Libvirt External Snapshots
description: How external snapshot layers map to disk states, and how to avoid snapshots that will not restore.
tags:
  - libvirt
  - qemu
  - snapshots
---

## Overview

This page covers how external snapshot layers are laid out on libvirt VMs and what practices keep snapshots restorable. For a snapshot that already fails to restore, see the external snapshot recovery page.

## How the Layers Line Up

When libvirt takes an external snapshot, it freezes the current disk file. It then creates a new overlay named after the snapshot. The overlay receives every write made after the snapshot.

After two snapshots the chain looks like this:

```text
base.qcow2                 (original disk)
  +- vm.Snap1              (overlay created by Snap1, holds writes from Snap1 until Snap2)
       +- vm.Snap2         (overlay created by Snap2, holds writes made after Snap2)
```

The disk state at the moment of snapshot X is the backing file of the overlay named X. That is the file that was active right before X was taken.

To restore Snap2, `vm.Snap1` is used. If `vm.Snap2` is layered on instead, the disk as it is now is gotten, including everything written after the snapshot.

Timestamps confirm this: the frozen file's modification time matches the `-mem` file's timestamp. The newer overlay has a later one.

## Preventing Restore Failures

A live snapshot of a VM with a virtiofs device fails to restore. These options are picked from, roughly from safest to most convenient:

1. **Shut the VM down before snapshotting.** An offline snapshot has no memory file and restores cleanly.
2. **Use disk-only snapshots.** No RAM or device state gets saved, so nothing can fail on device state:
   ```bash
   virsh snapshot-create-as <vm> <snap> --disk-only --atomic
   ```
   These are crash-consistent. The `--quiesce` flag is added when the QEMU Guest Agent is installed, which makes them filesystem-consistent.
3. **Detach the virtiofs device before a live snapshot** and reattach it afterwards. This is done when the RAM state is needed.
4. **Skip overlay chains.** Internal qcow2 snapshots, `qemu-img convert` copies of a shut-down VM, or backups from inside the guest are used instead.

## Keeping Chains Short

Every layer slows I/O slightly and adds a way for the chain to break. External snapshot chains aren't let grow long. They're flattened periodically, following the flatten section of the external snapshot recovery page.

For background on the qcow2 format, see the Virtual Disks Reference.
