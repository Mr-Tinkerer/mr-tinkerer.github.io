---
title: "Pt 2, Converting VMDK to QCOW2"
description: Converting the VMware base disk to QCOW2, building a native backing chain, and validating the images.
tags:
  - libvirt
  - vmware
  - qemu
  - migration
---

# Prerequisites

- [[Pt 1, Inventorying-the-VMware-source]]

---

## Convert the base descriptor

I convert the descriptor. QEMU reads the extent layout through it.

```bash
qemu-img convert -p -f vmdk -O qcow2 VMNAME.vmdk VMNAME-base.qcow2
```

Then I check the result:

```bash
qemu-img check VMNAME-base.qcow2
```

---

## Converting a child does not make a chain

A plain conversion of a snapshot descriptor makes a standalone [[Libvirt-glossary#QCOW2|QCOW2]]:

```bash
qemu-img convert -p -f vmdk -O qcow2 VMNAME-000001.vmdk VMNAME-child.qcow2
```

It holds the guest-visible contents of that VMDK chain. It does not create a [[Libvirt-glossary#Backing file and backing chain|backing file]] link to the converted base. I can end up with two large images holding duplicated data:

```text
base.qcow2              19.1 GiB
child.qcow2             19.1 GiB
```

---

## Build a native backing chain

When I want a real chain, I create the child explicitly after converting the base:

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

The child starts with only QCOW2 metadata and stores future changes. I inspect it:

```bash
qemu-img info --backing-chain VMNAME-snapshot.qcow2
```

The output must show `backing file: VMNAME-base.qcow2` and `backing file format: qcow2`.

---

## Validate every converted image

```bash
qemu-img check VMNAME-base.qcow2
qemu-img check VMNAME-snapshot.qcow2
```

To compare two independently converted states:

```bash
qemu-img compare VMNAME-base.qcow2 VMNAME-other-state.qcow2
```

`Images are identical.` means the guest-visible contents match. This is useful for a VMware snapshot I took while the VM was powered off and never booted again.

---

## Keep the chain together

A child with a relative backing reference needs its parent beside it:

```text
/var/lib/libvirt/images/
├── VMNAME-base.qcow2
└── VMNAME-snapshot.qcow2
```

Moving only the child breaks the reference until I repair it. After moving files, I verify again:

```bash
qemu-img info --backing-chain /var/lib/libvirt/images/VMNAME-snapshot.qcow2
```

For repairing a moved backing file, see the rebase entry in [[Libvirt-external-snapshot-recovery#Gotchas]].

---

## Verify

- [ ] I converted the base descriptor, not individual extents
- [ ] The base passes `qemu-img check`
- [ ] `qemu-img info --backing-chain` shows the intended relationships
- [ ] Parent and child files with relative references sit together
- [ ] I did not flatten anything by accident
- [ ] I kept the original VMware-derived conversion until the whole migration is done

---

# Next

[[Pt 3, Importing-the-disk-into-libvirt]]
