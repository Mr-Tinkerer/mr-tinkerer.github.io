---
title: VMware Migration Pt 2, Converting VMDK to QCOW2
description: Converting the VMware base disk to QCOW2, building a native backing chain, and validating the images.
tags:
  - libvirt
  - vmware
  - qemu
  - migration
---

## Overview

This step covers converting the VMware base disk to QCOW2, building a native backing chain, and validating the images.

## Prerequisites

- [[Pt 1, Inventorying-the-VMware-source]]

## Convert the Base Descriptor

The descriptor is converted. QEMU reads the extent layout through it.

```bash
qemu-img convert -p -f vmdk -O qcow2 VMNAME.vmdk VMNAME-base.qcow2
```

Then the result is checked:

```bash
qemu-img check VMNAME-base.qcow2
```

## Converting a Child Does Not Make a Chain

A plain conversion of a snapshot descriptor makes a standalone QCOW2:

```bash
qemu-img convert -p -f vmdk -O qcow2 VMNAME-000001.vmdk VMNAME-child.qcow2
```

It holds the guest-visible contents of that VMDK chain. It doesn't create a backing file link to the converted base. Two large images holding duplicated data can end up:

```text
base.qcow2              19.1 GiB
child.qcow2             19.1 GiB
```

## Build a Native Backing Chain

When a real chain is wanted, the child is created explicitly after converting the base:

```bash
qemu-img create -f qcow2 -F qcow2 -b VMNAME-base.qcow2 VMNAME-snapshot.qcow2
```

```text
VMNAME-snapshot.qcow2
        |
        | backing file
        v
VMNAME-base.qcow2
```

The child starts with only QCOW2 metadata and stores future changes. It's inspected:

```bash
qemu-img info --backing-chain VMNAME-snapshot.qcow2
```

The output must show `backing file: VMNAME-base.qcow2` and `backing file format: qcow2`.

## Validate Every Converted Image

```bash
qemu-img check VMNAME-base.qcow2
qemu-img check VMNAME-snapshot.qcow2
```

To compare two independently converted states:

```bash
qemu-img compare VMNAME-base.qcow2 VMNAME-other-state.qcow2
```

"Images are identical" means the guest-visible contents match. This is useful for a VMware snapshot taken while the VM was powered off and never booted again.

## Keep the Chain Together

A child with a relative backing reference needs its parent beside it:

```text
/var/lib/libvirt/images/
├── VMNAME-base.qcow2
└── VMNAME-snapshot.qcow2
```

Moving only the child breaks the reference until it's repaired. After moving files, verification is done again:

```bash
qemu-img info --backing-chain /var/lib/libvirt/images/VMNAME-snapshot.qcow2
```

For repairing a moved backing file, see the rebase entry in the external snapshot recovery gotchas.

## Verify

- [ ] The base descriptor was converted, not individual extents
- [ ] The base passes `qemu-img check`
- [ ] `qemu-img info --backing-chain` shows the intended relationships
- [ ] Parent and child files with relative references sit together
- [ ] Nothing was flattened by accident
- [ ] The original VMware-derived conversion was kept until the whole migration is done

---

Next: [[Pt 3, Importing-the-disk-into-libvirt]]
