---
title: libvirt external snapshot recovery
description: Restoring disk state when a live external snapshot will not revert, then cleaning up the snapshots and flattening to one image.
tags:
  - libvirt
  - qemu
  - snapshots
  - troubleshooting
---

This page covers a failed revert of a live [[Libvirt-glossary#External snapshot|external snapshot]]. It restores the disk state without the memory state, removes the leftover snapshot records, and flattens the result. For how the layers map to disk states, read [[Libvirt-external-snapshots]] first.

Placeholders:

- `<vm>`: the libvirt domain name, from `virsh list --all`
- `<dir>`: the directory holding the disk images
- `<snap>`: the snapshot I want to go back to

I run commands with `sudo` when the VM lives in the system libvirt instance.

---

## Symptoms and cause

Reverting in [[Libvirt-glossary#Virt Manager|Virt Manager]] or with `virsh snapshot-revert` fails with an error like this:

```text
operation failed: job 'snapshot' failed: load of migration failed: Invalid argument:
error while loading state for instance 0x0 of device '.../vhost-user-fs':
Loading VM subsection 'vhost-user-fs-backend' in 'vhost-user-fs' failed:
Failed to load element of type virtio-fs back-end state for back-end
```

The snapshot was taken while the VM was running, so it includes a RAM and device-state file named `<vm>-mem.<snap>`. That state includes the [[Libvirt-glossary#virtiofs|virtiofs]] backend, which QEMU generally cannot reload. The disk data is fine. Only the RAM state is unrestorable.

The workaround costs me the running state: open programs and in-memory data. The guest boots as if power was cut, which NTFS, ext4, and xfs normally handle.

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

The `backing file:` line of the overlay named `<snap>` is the disk state I want. `-U` (`--force-share`) lets `qemu-img info` read images that are in use.

After a failed revert, `domblklist` can point at a frozen layer instead of the newest overlay. I do not boot the VM until I fix that. Booting would write into a file that other snapshots depend on.

---

## Back up first

Overlays and backing files depend on each other, so I copy all of them before changing anything.

```bash
virsh destroy <vm>          # only if it's running; skip if it's already off
mkdir -p /path/to/backup
cp --reflink=auto "<dir>"/<vm>* /path/to/backup/
```

I adjust the glob so it catches every image file for the VM, including `-mem` files if I want them.

---

## Restore the disk state without memory

1. **Create a fresh [[Libvirt-glossary#Overlay|overlay]] on the frozen file.** I never boot directly from the frozen backing file.
   ```bash
   cd "<dir>"
   qemu-img create -f qcow2 \
     -b "<frozen-file-from-diagnose>" -F qcow2 \
     <vm>.restored.qcow2
   chown libvirt-qemu:libvirt-qemu <vm>.restored.qcow2
   ```
   `-F` takes the backing file's real format, which `qemu-img info` shows. The owner differs by distro (`libvirt-qemu`, `qemu`, or `root`), so I match my other images.

2. **Point the VM at it.** Run `virsh edit <vm>` and change the disk's `<source file='...'/>` to the full path of `<vm>.restored.qcow2`.

3. **Boot and verify.**
   ```bash
   virsh start <vm>
   ```
   Windows sees an unclean shutdown and recovers on its own. I check that the data from that point in time is present, then run a filesystem check (`chkdsk` on Windows, `fsck` on Linux).

If the expected changes are missing, I picked the wrong layer. I try the next one up the chain, since I have backups.

---

## Clean up the snapshot metadata

`virsh snapshot-delete <vm> <snap>` fails for external snapshots. `--metadata` removes only libvirt's records and never touches disk files. I delete children first, then parents.

```bash
virsh snapshot-delete <vm> <child-snap> --metadata
virsh snapshot-delete <vm> <snap> --metadata
virsh snapshot-list <vm>            # should be empty
```

`/var/lib/libvirt/qemu/snapshot/<vm>/` should have no XML files left for this VM.

---

## Flatten to a single image

[[Libvirt-glossary#Flattening|Flattening]] removes every dependency on the old overlays. I recommend it after a recovery.

1. Shut the VM down completely.
2. Verify the chain I am about to flatten:
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
6. Boot and test. I delete the old chain files and my backups only after the VM has been stable for a while.

Renaming the flat image and updating the path with `virsh edit` is cosmetic.

---

## Verify

- [ ] `virsh domblklist <vm>` shows the flat (or restored) image, and the file exists
- [ ] `qemu-img info` shows no backing file, if I flattened
- [ ] `qemu-img check` reports no errors
- [ ] `virsh snapshot-list <vm>` is empty
- [ ] The VM boots, the data is as expected, and the guest filesystem check is clean
- [ ] `ls -l` shows ownership QEMU can open
- [ ] Backups are deleted only after all of the above passes and I have used the VM for a while

---

## Gotchas

| Problem | Likely cause and fix |
|---|---|
| VM will not start after `virsh edit`, "Could not open ... Permission denied" | Wrong owner or permissions on the new image, or AppArmor/SELinux. I check `ls -l` against my other images. |
| "Could not open backing file" | The overlay's backing path is wrong, or the file moved or was deleted. I check with `qemu-img info`. If I only moved the file, `qemu-img rebase -u -b <path> -F qcow2 <overlay>` repairs the path. |
| Restored VM is missing recent changes | I layered on the wrong file. I use the backing file of the overlay named after the snapshot. |
| Restored VM has extra changes | I layered on the overlay instead of its backing file. |
| `snapshot-delete` fails without `--metadata` | Expected for external snapshots. I use `--metadata`, then handle the files by hand. |
| `qemu-img info` says "Failed to get shared write lock" | The image is in use. I use `-U` or shut the VM down. |
| Flattened image is much larger than expected | Normal, because the flat image holds all chain data. I recover space with `qemu-img convert -c` or by trimming the guest. See [[Virtual-disks-reference#10. Reclaiming space from the host]]. |
