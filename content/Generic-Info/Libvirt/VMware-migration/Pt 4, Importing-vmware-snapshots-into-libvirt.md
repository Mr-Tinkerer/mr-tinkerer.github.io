---
title: VMware Migration Pt 4, Importing VMware Snapshots into libvirt
description: Recreating an offline VMware snapshot as a named libvirt snapshot without depending on the VMware-derived image.
tags:
  - libvirt
  - vmware
  - snapshots
  - migration
---

## Overview

This step covers recreating an offline VMware snapshot as a named libvirt snapshot without depending on the VMware-derived image.

## Prerequisites

- [[Pt 1, Inventorying-the-VMware-source]]
- [[Pt 2, Converting-vmdk-to-qcow2]]
- [[Pt 3, Importing-the-disk-into-libvirt]]

## Two Separate Tasks

Keeping a VMware snapshot as a QEMU backing layer and showing it as a named snapshot in Virt Manager are separate. A working chain doesn't create a libvirt snapshot. `virsh snapshot-list VMNAME` can list nothing while the chain works. The named snapshot needs its own snapshot metadata. The disks and the metadata are verified independently.

## Define the Snapshot

This example is for a VMware snapshot taken while the VM was powered off, with no writes afterward. It's illustrative, not a maintained file.

```xml
<domainsnapshot>
  <name>Snapshot-Name</name>
  <description>Imported VMware snapshot</description>
  <creationTime>...</creationTime>
  <state>shutoff</state>
  <memory snapshot="no" />

  <disks>
    <disk name="sda" snapshot="external">
      <driver type="qcow2" />
      <source file="/path/to/libvirt-managed-snapshot-layer" />
    </disk>
  </disks>

  <domain type="kvm">
    <!-- the complete relevant domain definition, including the disk below -->
    <devices>
      <disk type="file" device="disk">
        <driver name="qemu" type="qcow2" />
        <source file="/path/to/base.qcow2" />
        <target dev="sda" bus="sata" />
      </disk>
    </devices>
  </domain>
</domainsnapshot>
```

How each part is filled:

- **`<disks>`** — Describes the external storage layer libvirt manages for the snapshot
- **`<domain>`** — Describes the domain definition the snapshot represents, including its disk. For an offline snapshot where the VM configuration didn't change, the current inactive domain XML works. The full definition is included, not only the name and UUID
- **`<creationTime>`** — libvirt requires it when a snapshot is redefined. It can be rebuilt from VMware metadata. If the exact time doesn't matter, any valid timestamp works, and the displayed date won't match the original
- **`<state>`** — Is `shutoff` for a snapshot taken powered off
- **`<memory snapshot="no"/>`** — Fits a powered-off snapshot with no memory state. A running VMware snapshot's memory state in the `.vmsn` can't be swapped into a libvirt memory snapshot

## Use a Distinct Layer File

The VMware-derived child isn't registered as the libvirt snapshot's external disk. Take this arrangement:

```text
VMNAME-base.qcow2
        |
        v
VMNAME-vmware-snapshot.qcow2
```

If the snapshot definition names `VMNAME-vmware-snapshot.qcow2` as its external disk, a later restore can make libvirt create a new image backed by that child. If the child is removed or renamed, the VM fails to start.

Instead, when the base equals the snapshot-time state, the base image is put in the snapshot's stored domain definition. A new, distinct filename is reserved for the layer libvirt manages, such as `VMNAME.Setup-the-Snapshot`:

```text
VMNAME.Setup-the-Snapshot
        |
        v
VMNAME-base.qcow2
```

The full layout kept:

```text
/path/to/libvirt/images/
│
├── VMNAME-base.qcow2
│
├── VMNAME-vmware-snapshot.qcow2
│   └── backing: VMNAME-base.qcow2
│
└── VMNAME.Setup-the-Snapshot
    └── backing: VMNAME-base.qcow2
```

The VMware-derived child stays as a migration artifact. The domain still has one virtual disk. Not every file is attached at once. The original conversion is kept until the migration is fully tested.

## Redefine and Check

The XML is validated:

```bash
virt-xml-validate snapshot.xml
```

The snapshot is redefined:

```bash
sudo virsh snapshot-create VMNAME snapshot.xml --redefine --current
```

What libvirt recorded is checked:

```bash
sudo virsh snapshot-list VMNAME
sudo virsh snapshot-list VMNAME --tree
sudo virsh snapshot-dumpxml VMNAME Snapshot-Name
```

A snapshot in the list doesn't prove restore works.

## Test a Real Restore

1. Boot the VM
2. Make a harmless, identifiable change in the guest
3. Shut the VM down
4. Restore the imported snapshot
5. Boot again
6. Confirm the change is gone
7. Inspect the resulting chain:

```bash
qemu-img info --backing-chain /path/to/resulting-image.qcow2
```

The chain should show the intended backing files and no corruption:

```text
image: VMNAME.Setup-the-Snapshot
file format: qcow2
backing file: VMNAME-base.qcow2
backing file format: qcow2
...
image: VMNAME-base.qcow2
file format: qcow2
```

The real files are inspected with `ls -lh` and `qemu-img check` too, not only the snapshot tree. If the chain references a file removed on purpose, the snapshot definition is unusable and it's fixed before more testing.

## Gotchas

- Registering the VMware-derived child as the external disk makes restore create an image that depends on it
- The snapshot tree can look correct while restore is broken. Only the create-change-restore test catches it
- Snapshot ancestry isn't inferred from filenames. The `.vmsd` data, the VMDK parent and child links, the state at snapshot time, and whether writes happened afterward are used
- For several snapshots, the QEMU chain (snapshot3 over snapshot2 over snapshot1 over base) and the libvirt snapshot tree are separate structures. They're mapped by hand

## Verify

- [ ] `virt-xml-validate` passes
- [ ] The snapshot has a name and description
- [ ] State and memory setting suit an offline snapshot
- [ ] `<domain>` holds the full domain definition
- [ ] Snapshot disk metadata points at the libvirt-managed layer
- [ ] Nothing requires the VMware-derived child as a backing file
- [ ] `snapshot-list`, `--tree`, and `snapshot-dumpxml` show the expected data
- [ ] A restore test succeeds and the post-restore chain is correct
- [ ] Guest changes made after the snapshot disappear after restore
- [ ] The original VMware directory is kept until every state needed is tested
