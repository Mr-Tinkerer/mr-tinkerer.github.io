---
title: libvirt external snapshot recovery
description: Restoring disk state when a live external snapshot will not revert, then cleaning up the snapshots and flattening to one image.
tags:
  - libvirt
  - qemu
  - snapshots
  - troubleshooting
---

# libvirt external snapshot recovery

This page covers a failed revert of a live [[Libvirt/Libvirt-glossary#External snapshot|external snapshot]]. It restores the disk state without the memory state, removes the leftover snapshot records, and flattens the result. For how the layers map to disk states, read [[Libvirt/Libvirt-external-snapshots]] first.

Placeholders:

- `<vm>`: the libvirt domain name, from `virsh list --all`
- `<dir>`: the directory holding the disk images
- `<snap>`: the snapshot to revert to

Commands use `sudo` when the VM lives in the system libvirt instance.

---

## Symptoms and cause

Reverting in [[Libvirt/Libvirt-glossary#Virt Manager|Virt Manager]] or with `virsh snapshot-revert` fails with an error like this:

```text
operation failed: job 'snapshot' failed: load of migration failed: Invalid argument:
error while loading state for instance 0x0 of device '.../vhost-user-fs':
Loading VM subsection 'vhost-user-fs-backend' in 'vhost-user-fs' failed:
Failed to load element of type virtio-fs back-end state for back-end
```

The snapshot was taken while the VM was running, so it includes a RAM and device-state file named `<vm>-mem.<snap>`. That state includes the [[Libvirt/Libvirt-glossary#virtiofs|virtiofs]] backend, which QEMU generally cannot reload. The disk data is fine. Only the RAM state is unrestorable.

The workaround costs the running state: open programs and in-memory data. The guest boots as if power was cut, which NTFS, ext4, and xfs normally handle.

---

## Diagnose

These commands are read-only.

```bash
# Snapshot tree and what the VM currently uses
virsh snapshot-list <vm> --tree
virsh domblklist <vm>

# Inspect the chain of the overlay named after the snapshot
cd "<dir>"
qemu-img info --backing-chain -U "<vm>.<snap>"
```

The `backing file:` line of the overlay named `<snap>` is the disk state to restore. `-U` (`--force-share`) lets `qemu-img info` read images that are in use.

After a failed revert, `domblklist` can point at a frozen layer instead of the newest overlay. The VM must not boot until that's fixed. Booting would write into a file that other snapshots depend on.

---

## Back up first

Overlays and backing files depend on each other, so all of them are copied before making changes.

```bash
virsh destroy <vm>          # only if it's running; skip if it's already off
mkdir -p /path/to/backup
cp --reflink=auto "<dir>"/<vm>* /path/to/backup/
```

The glob is adjusted to catch every image file for the VM, including `-mem` files if desired.

---

## Restore the disk state without memory

1. **Create a fresh [[Libvirt/Libvirt-glossary#Overlay|overlay]] on the frozen file.** The frozen backing file never boots directly.
   ```bash
   cd "<dir>"
   qemu-img create -f qcow2 \
     -b "<frozen-file-from-diagnose>" -F qcow2 \
     <vm>.restored.qcow2
   chown libvirt-qemu:libvirt-qemu <vm>.restored.qcow2
   ```
   `-F` takes the backing file's real format, which `qemu-img info` shows. The owner differs by distro (`libvirt-qemu`, `qemu`, or `root`), so it matches other images.

2. **Point the VM at it.** Run `virsh edit <vm>` and change the disk's `<source file='...'/>` to the full path of `<vm>.restored.qcow2`.

3. **Boot and verify.**
   ```bash
   virsh start <vm>
   ```
   Windows sees an unclean shutdown and recovers on its own. Data from that point in time should be present; run a filesystem check (`chkdsk` on Windows, `fsck` on Linux).

If expected changes are missing, the wrong layer was chosen. The next layer up the chain is tried, since backups exist.

---

## Clean up the snapshot metadata

`virsh snapshot-delete <vm> <snap>` fails for external snapshots. `--metadata` removes only libvirt's records and never touches disk files. Children are deleted first, then parents.

```bash
virsh snapshot-delete <vm> <child-snap> --metadata
virsh snapshot-delete <vm> <snap> --metadata
virsh snapshot-list <vm>            # should be empty
```

`/var/lib/libvirt/qemu/snapshot/<vm>/` should have no XML files left for this VM.

---

## Flatten to a single image

[[Libvirt/Libvirt-glossary#Flattening|Flattening]] removes every dependency on the old overlays. It is recommended after a recovery.

1. Shut the VM down completely.
2. Verify the chain about to be flattened:
   ```bash
   qemu-img info --backing-chain <vm>.restored.qcow2
   ```
3. Convert. This reads the whole chain and writes one independent image. `-c` compresses, at the cost of write speed.
   ```bash
   qemu-img convert -p -O qcow2 <vm>.restored.qcow2 <vm>.flat.qcow2
   ```
4. Run `virsh edit <vm>` to point at the flat image, then fix ownership and permissions:
   ```bash
   chown libvirt-qemu:libvirt-qemu <vm>.flat.qcow2
   chmod 660 <vm>.flat.qcow2
   ```
5. Verify it is standalone and healthy:
   ```bash
   qemu-img info <vm>.flat.qcow2      # no "backing file:" line
   qemu-img check <vm>.flat.qcow2     # no errors
   ```
6. Boot and test. Old chain files and backups are deleted only after the VM has been stable for a while.

Renaming the flat image and updating the path with `virsh edit` is cosmetic.

---

## Verify

- [ ] `virsh domblklist <vm>` shows the flat (or restored) image, and the file exists
- [ ] `qemu-img info` shows no backing file, if flattened
- [ ] `qemu-img check` reports no errors
- [ ] `virsh snapshot-list <vm>` is empty
- [ ] The VM boots, the data is as expected, and the guest filesystem check is clean
- [ ] `ls -l` shows ownership QEMU can open
- [ ] Backups are deleted only after all of the above passes and the VM has been used for a while

---

## Gotchas

| Problem | Likely cause and fix |
|---|---|
| VM will not start after `virsh edit`, "Could not open ... Permission denied" | Wrong owner or permissions on the new image, or AppArmor/SELinux. `ls -l` is checked against other images. |
| "Could not open backing file" | The overlay's backing path is wrong, or the file moved or was deleted. `qemu-img info` is checked. If the file was only moved, `qemu-img rebase -u -b <path> -F qcow2 <overlay>` repairs the path. |
| Restored VM is missing recent changes | The wrong file was layered. The backing file of the overlay named after the snapshot is used. |
| Restored VM has extra changes | The overlay was layered instead of its backing file. |
| `snapshot-delete` fails without `--metadata` | Expected for external snapshots. `--metadata` is used, then files are handled by hand. |
| `qemu-img info` says "Failed to get shared write lock" | The image is in use. `-U` is used or the VM is shut down. |
| Flattened image is much larger than expected | Normal, because the flat image holds all chain data. Space is recovered with `qemu-img convert -c` or by trimming the guest. See [[Virtual-disks-reference#10. Reclaiming space from the host]]. |
