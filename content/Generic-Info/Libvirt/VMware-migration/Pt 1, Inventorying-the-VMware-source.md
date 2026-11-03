---
title: VMware Migration Pt 1, Inventorying the VMware Source
description: Protecting the original VMware VM and mapping its disks, snapshots, and hardware before converting anything.
tags:
  - libvirt
  - vmware
  - migration
---

## Overview

This guide covers migrating VMware VMs to QEMU/KVM/libvirt by hand, inspecting every step. The process is the same for Windows and Linux VMs. The original VMware files are kept untouched, VMDK disks are converted to QCOW2, and useful snapshot states are preserved. Automated one-to-one migration tools aren't used.

## Prerequisites

None — this is the first step.

## Goal

VMware VMs are migrated to QEMU/KVM/libvirt on the Gigabyte laptop by hand. The original VMware files are kept untouched, VMDK disks are converted to QCOW2, and useful snapshot states are preserved.

## Protect the Source

The original VMware VM directory is backed up before anything is touched. Work is done from a copy or from the backup. While experimenting, the original VMDKs are not modified, renamed, deleted, or consolidated.

## Know the Files

A VMware VM directory can contain:

- **`.vmx`** — Hardware and configuration. Used as a reference when building the libvirt VM.
- **`.vmdk` descriptor files, plus extent files** — Such as `disk-s001.vmdk` and `disk-s002.vmdk`
- **`.vmsd`** — Snapshot names, IDs, relationships, and the disk descriptors tied to each snapshot. Helps rebuild the snapshot history. Not a libvirt snapshot database.
- **`.vmsn`** — Snapshot state. For a running VM it holds memory state. Not a portable libvirt snapshot file. For a powered-off VM, the disk state is rebuilt from the VMDK relationships and the `.vmsd` data.
- **`.nvram`** — EFI firmware state. Decided separately whether it needs carrying over. Not copied into the new VM by default.
- **Lock directories and files**

See the VMware VM files glossary entry for details.

## Split VMDKs Are One Disk

A descriptor such as `VMNAME.vmdk` can reference many extent files:

```text
VMNAME-s001.vmdk
VMNAME-s002.vmdk
...
VMNAME-s021.vmdk
```

Together those extents form one virtual disk. They're not separate disks or separate snapshots. The descriptor is converted, never the individual extents.

## Inventory Commands

```bash
ls -lah
```

Inspect the VMX:

```bash
grep -E '^(firmware|numvcpus|cpuid|memsize|.*\.fileName|.*\.present|.*\.virtualDev|.*\.generatedAddress)' VMNAME.vmx
```

Inspect the VMDK relationships:

```bash
qemu-img info --backing-chain VMNAME.vmdk
```

For a suspected snapshot descriptor:

```bash
qemu-img info VMNAME-000001.vmdk
```

If the output says `backing file: VMNAME.vmdk`, the descriptor is a child of the base disk.

### Questions to Answer

Before continuing, these questions must be answered:

- Which descriptor is the base disk?
- Which descriptor is currently active?
- What backing file does each snapshot descriptor reference?
- How many VMware snapshots exist?
- Which disk state belongs to each snapshot?
- Does the VM currently point at a snapshot (redo-log) descriptor?
- Was each snapshot taken powered off or while running?

## Snapshot-Time State Versus Current State

If a VM is shut down, a snapshot is taken, and never booted again, no writes happen after the snapshot. Then snapshot state, base disk state, and current state are identical.

If the VM is booted and modified after the snapshot, the active child holds post-snapshot changes. Converting the active descriptor then doesn't give the snapshot-time state. The `.vmsd` data and the backing relationships are used to decide which disk state belongs to each snapshot. Snapshot ancestry isn't inferred from filenames.

## Verify

- [ ] Original VMware directory is backed up and unmodified
- [ ] `.vmx`, `.vmsd`, and `.vmsn` are inspected
- [ ] Base VMDK and snapshot or child VMDKs are identified
- [ ] Parent and child relationships are verified with `qemu-img info`

---

Next: [[Pt 2, Converting-vmdk-to-qcow2]]
