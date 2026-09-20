---
title: "Pt 4, Importing VMware snapshots into libvirt"
description: Recreating an offline VMware snapshot as a named libvirt snapshot without depending on the VMware-derived image.
tags:
  - libvirt
  - vmware
  - snapshots
  - migration
---

# Prerequisites

- [[Pt 1, Inventorying-the-VMware-source]]
- [[Pt 2, Converting-vmdk-to-qcow2]]
- [[Pt 3, Importing-the-disk-into-libvirt]]

---

## Two separate tasks

Keeping a VMware snapshot as a [[Libvirt-glossary#Backing file and backing chain|QEMU backing layer]] and showing it as a named snapshot in Virt Manager are separate. A working chain does not create a libvirt snapshot. `virsh snapshot-list VMNAME` can list nothing while the chain works. The named snapshot needs its own [[Libvirt-glossary#libvirt snapshot metadata|snapshot metadata]]. I verify the disks and the metadata independently.

---

## Define the snapshot

This example is for a VMware snapshot I took while the VM was powered off, with no writes afterward. It is illustrative, not a maintained file.

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

How I fill each part:

- **`<disks>`** describes the external storage layer libvirt manages for the snapshot.
- **`<domain>`** describes the domain definition the snapshot represents, including its disk. For an offline snapshot where the VM configuration did not change, the current inactive domain XML works. I include the full definition, not only the name and UUID.
- **`<creationTime>`** libvirt requires it when I redefine a snapshot. I can rebuild it from VMware metadata. If the exact time does not matter, any valid timestamp works, and the displayed date will not match the original.
- **`<state>`** is `shutoff` for a snapshot taken powered off.
- **`<memory snapshot="no"/>`** fits a powered-off snapshot with no memory state. A running VMware snapshot's memory state in the `.vmsn` cannot be swapped into a libvirt memory snapshot.

---

## Use a distinct layer file

I do not register the VMware-derived child as the libvirt snapshot's external disk. Take this arrangement:

```text
VMNAME-base.qcow2
        |
        v
VMNAME-vmware-snapshot.qcow2
```

If the snapshot definition names `VMNAME-vmware-snapshot.qcow2` as its external disk, a later restore can make libvirt create a new image backed by that child. If I removed or renamed the child, the VM fails to start.

Instead, when the base equals the snapshot-time state, I put the base image in the snapshot's stored domain definition. I reserve a new, distinct filename for the layer libvirt manages, such as `VMNAME.Setup-the-Snapshot`:

```text
VMNAME.Setup-the-Snapshot
        |
        v
VMNAME-base.qcow2
```

The full layout I keep:

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

The VMware-derived child stays as a migration artifact. The domain still has one virtual disk. I do not attach every file at once. I keep the original conversion until the migration is fully tested.

---

## Redefine and check

Validate the XML:

```bash
virt-xml-validate snapshot.xml
```

Redefine the snapshot:

```bash
sudo virsh snapshot-create VMNAME snapshot.xml --redefine --current
```

Check what libvirt recorded:

```bash
sudo virsh snapshot-list VMNAME
sudo virsh snapshot-list VMNAME --tree
sudo virsh snapshot-dumpxml VMNAME Snapshot-Name
```

A snapshot in the list does not prove restore works.

---

## Test a real restore

1. Boot the VM.
2. Make a harmless, identifiable change in the guest.
3. Shut the VM down.
4. Restore the imported snapshot.
5. Boot again.
6. Confirm the change is gone.
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

Inspect the real files with `ls -lh` and `qemu-img check` too, not only the snapshot tree. If the chain references a file I removed on purpose, the snapshot definition is unusable and I fix it before more testing.

---

## Gotchas

- Registering the VMware-derived child as the external disk makes restore create an image that depends on it. See [[Pt 4, Importing-vmware-snapshots-into-libvirt#Use a distinct layer file]].
- The snapshot tree can look correct while restore is broken. Only the create-change-restore test catches it.
- Do not infer snapshot ancestry from filenames. Use the `.vmsd` data, the VMDK parent and child links, the state at snapshot time, and whether writes happened afterward.
- For several snapshots, the QEMU chain (`snapshot3` over `snapshot2` over `snapshot1` over `base`) and the libvirt snapshot tree are separate structures. I map them by hand.

---

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
- [ ] I keep the original VMware directory until every state I need is tested
