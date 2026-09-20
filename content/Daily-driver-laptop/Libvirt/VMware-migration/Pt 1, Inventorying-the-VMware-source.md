---
title: "Pt 1, Inventorying the VMware source"
description: Protecting the original VMware VM and mapping its disks, snapshots, and hardware before converting anything.
tags:
  - libvirt
  - vmware
  - migration
---

# Prerequisites

None — this is the first step.

---

## Goal

I migrate VMware VMs to QEMU/KVM/libvirt on my [[Gigabyte-gaming-a16-ga6h-overview|Gigabyte laptop]] by hand so I can inspect every step. The process is the same for Windows and Linux VMs. I keep the original VMware files untouched, convert [[Libvirt-glossary#VMDK|VMDK]] disks to [[Libvirt-glossary#QCOW2|QCOW2]], and keep the useful snapshot states. I do not use an automated one-to-one migration tool.

---

## Protect the source

I back up the original VMware VM directory before I touch anything. I work from a copy or from the backup. While I experiment, I do not modify, rename, delete, or consolidate the original VMDKs.

---

## Know the files

A VMware VM directory can contain the following. See [[Libvirt-glossary#VMware VM files|VMware VM files]] for the details.

- `.vmx`: hardware and configuration. I use it as a reference when I build the libvirt VM in [[Pt 3, Importing-the-disk-into-libvirt]].
- `.vmdk` descriptor files, plus extent files such as `disk-s001.vmdk` and `disk-s002.vmdk`.
- `.vmsd`: snapshot names, IDs, relationships, and the disk descriptors tied to each snapshot. It helps me rebuild the snapshot history. It is not a libvirt snapshot database.
- `.vmsn`: snapshot state. For a running VM it holds memory state. It is not a portable libvirt snapshot file. For a powered-off VM I rebuild the disk state from the VMDK relationships and the `.vmsd` data.
- `.nvram`: EFI firmware state. I decide separately whether it needs carrying over. I do not copy it into the new VM by default.
- Lock directories and files.

---

## Split VMDKs are one disk

A descriptor such as `VMNAME.vmdk` can reference many extent files:

```text
VMNAME-s001.vmdk
VMNAME-s002.vmdk
...
VMNAME-s021.vmdk
```

Together those extents form one virtual disk. They are not separate disks or separate snapshots. I convert the descriptor in [[Pt 2, Converting-vmdk-to-qcow2]] and never the individual extents.

---

## Inventory commands

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

I need to answer these questions before I continue:

- Which descriptor is the base disk?
- Which descriptor is currently active?
- What backing file does each snapshot descriptor reference?
- How many VMware snapshots exist?
- Which disk state belongs to each snapshot?
- Does the VM currently point at a snapshot (redo-log) descriptor?
- Was each snapshot taken powered off or while running?

---

## Snapshot-time state versus current state

If I shut a VM down, take a snapshot, and never boot it again, no writes happen after the snapshot. Then snapshot state, base disk state, and current state are identical.

If I boot and modify the VM after the snapshot, the active child holds post-snapshot changes. Converting the active descriptor then does not give me the snapshot-time state. I use the `.vmsd` data and the backing relationships to decide which disk state belongs to each snapshot. I do not infer snapshot ancestry from filenames.

---

## Verify

- [ ] Original VMware directory is backed up and unmodified
- [ ] `.vmx`, `.vmsd`, and `.vmsn` are inspected
- [ ] Base VMDK and snapshot or child VMDKs are identified
- [ ] Parent and child relationships are verified with `qemu-img info`

---

# Next

[[Pt 2, Converting-vmdk-to-qcow2]]
