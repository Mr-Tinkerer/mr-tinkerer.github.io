---
title: "Pt 3, Importing the disk into libvirt"
description: Creating the libvirt VM from the converted QCOW2, matching the VMware hardware conservatively, and swapping the guest tools.
tags:
  - libvirt
  - vmware
  - windows
  - linux
  - migration
---

# Prerequisites

- [[Pt 1, Inventorying-the-VMware-source]]
- [[Pt 2, Converting-vmdk-to-qcow2]]

---

## Point the VM at the right image

I create the VM in [[Libvirt-glossary#Virt Manager|Virt Manager]] using an existing disk image. For a normal active chain, I select the top image:

```text
VM
 |
 v
VMNAME-active.qcow2
 |
 v
VMNAME-base.qcow2
```

The backing file does not get attached as a second disk. QEMU follows the [[Libvirt-glossary#Backing file and backing chain|backing chain]] internally. The domain has one virtual disk.

Putting both files in a libvirt storage pool only makes them storage volumes. It does not create libvirt snapshots. I cover that in [[Pt 4, Importing-vmware-snapshots-into-libvirt]].

---

## Match the hardware conservatively

For an existing VM, I choose compatibility over optimization at first. From the `.vmx` I record:

- firmware type (EFI or BIOS)
- vCPU count and RAM
- disk controller and network adapter type
- MAC addresses, if I need to keep them
- CD/DVD device and any other device the guest depends on

I do not switch the storage controller to VirtIO on a guest that lacks the drivers. I start with a widely supported emulated controller and network adapter, and I move storage and network devices to VirtIO one at a time after the guest boots. The controller options, including the VMware PVSCSI bridge, are in [[Virtual-disks-reference#3. SCSI controllers]].

For UEFI guests I also check OVMF firmware, Secure Boot needs, NVRAM variables, and machine type. My notes on firmware and chipset are in [[Libvirt-Information#VM Firmware]] and [[Libvirt-Information#Chipset]]. Windows can detect changed virtual hardware as new devices. I change one major hardware characteristic at a time so I can tell what broke.

---

## Swap the guest tools

The process is the same for Windows and Linux guests. Only the guest tools differ. I uninstall VMware Tools and install the [[Libvirt-glossary#VirtIO drivers|VirtIO drivers]] (`virtio-win` on Windows).

The driver installation follows the same bootstrap as a physical-to-virtual move. I boot on SATA first, stage the drivers, then switch the bus. The steps are in [[Virtual-disks-reference#11. Physical ⇄ virtual migration (P2V / V2P)]]. If Windows still reports a missing storage driver afterward, see [[Scsi-drive-troubleshooting]].

---

## Verify

- [ ] Firmware, Secure Boot, and NVRAM are configured as the guest needs
- [ ] RAM and vCPU count match the VMware VM
- [ ] Disk controller is the conservative choice
- [ ] Network configuration works
- [ ] The VM boots
- [ ] VMware Tools is gone and the VirtIO drivers are installed

---

# Next

[[Pt 4, Importing-vmware-snapshots-into-libvirt]]
