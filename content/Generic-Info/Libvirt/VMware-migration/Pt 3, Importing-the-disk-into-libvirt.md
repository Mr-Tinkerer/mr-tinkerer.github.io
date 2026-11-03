---
title: VMware Migration Pt 3, Importing the Disk into libvirt
description: Creating the libvirt VM from the converted QCOW2, matching the VMware hardware conservatively, and swapping the guest tools.
tags:
  - libvirt
  - vmware
  - windows
  - linux
  - migration
---

## Overview

This step covers creating the libvirt VM from the converted QCOW2, matching the VMware hardware conservatively, and swapping the guest tools.

## Prerequisites

- [[Pt 1, Inventorying-the-VMware-source]]
- [[Pt 2, Converting-vmdk-to-qcow2]]

## Point the VM at the Right Image

The VM is created in Virt Manager using an existing disk image. For a normal active chain, the top image is selected:

```text
VM
 |
 v
VMNAME-active.qcow2
 |
 v
VMNAME-base.qcow2
```

The backing file doesn't get attached as a second disk. QEMU follows the backing chain internally. The domain has one virtual disk.

Putting both files in a libvirt storage pool only makes them storage volumes. It doesn't create libvirt snapshots. That's covered in Pt 4.

## Match the Hardware Conservatively

For an existing VM, compatibility over optimization is chosen at first. From the `.vmx`, these are recorded:

- Firmware type (EFI or BIOS)
- vCPU count and RAM
- Disk controller and network adapter type
- MAC addresses, if keeping them
- CD/DVD device and any other device the guest depends on

The storage controller isn't switched to VirtIO on a guest lacking the drivers. A widely supported emulated controller and network adapter are used first, then storage and network devices are moved to VirtIO one at a time after the guest boots. The controller options, including the VMware PVSCSI bridge, are in the Virtual Disks Reference.

For UEFI guests, OVMF firmware, Secure Boot needs, NVRAM variables, and machine type are also checked. Notes on firmware and chipset are in the Libvirt Information pages. Windows can detect changed virtual hardware as new devices. One major hardware characteristic is changed at a time so what broke can be told.

## Swap the Guest Tools

The process is the same for Windows and Linux guests. Only the guest tools differ. VMware Tools is uninstalled and the VirtIO drivers (`virtio-win` on Windows) are installed.

The driver installation follows the same bootstrap as a physical-to-virtual move. The guest boots on SATA first, stages the drivers, then switches the bus. The steps are in the Physical ⇄ Virtual migration page. If Windows still reports a missing storage driver afterward, the SCSI drive troubleshooting page is consulted.

## Verify

- [ ] Firmware, Secure Boot, and NVRAM are configured as the guest needs
- [ ] RAM and vCPU count match the VMware VM
- [ ] Disk controller is the conservative choice
- [ ] Network configuration works
- [ ] The VM boots
- [ ] VMware Tools is gone and the VirtIO drivers are installed

---

Next: [[Pt 4, Importing-vmware-snapshots-into-libvirt]]
