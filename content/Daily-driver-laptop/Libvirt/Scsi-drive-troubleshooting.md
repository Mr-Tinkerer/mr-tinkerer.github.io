---
title: Troubleshooting a Windows 10 VirtIO SCSI Drive
description: Fixing a secondary SCSI drive that Windows 10 wouldn't detect, or flagged with a Code 28/Code 10 error, under libvirt/KVM.
tags:
  - Virtual_Machines
  - QEMU
  - Windows
  - Troubleshooting
---
# Troubleshooting a Windows 10 VirtIO SCSI Drive

When I added a secondary SCSI drive to my Windows 10 VM, it wasn't detected, showed a yellow exclamation mark (Code 28/Code 10), or rejected the [[Libvirt-glossary#VirtIO drivers|VirtIO drivers]]. Here's how I resolved it, step by step.

## 1. Verify the SCSI controller model

Windows throws a **Code 10 (Device cannot start)** error if the underlying emulated controller is misconfigured.

1. Shut down the VM.
2. In [[Libvirt-glossary#Virt Manager|Virt Manager]], open the VM's details and select the **SCSI Controller** in the hardware panel.
3. Change the **Model** dropdown from `LSI Logic` (the default) to **`VirtIO SCSI`**.
4. Click **Apply** and start the VM.

Via `virsh edit`, the controller XML should read:
```xml
<controller type='scsi' index='0' model='virtio-scsi'/>
```

## 2. Get the VirtIO drivers ISO

I download the driver ISO (e.g. `virtio-win-0.1.285-1.iso`) from the [Fedora VirtIO archive](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/), then in Virt Manager add it as a CD-ROM device (or point an existing one at the downloaded `.iso`) and apply.

## 3. Install the VirtIO SCSI driver

Windows 10 doesn't include the VirtIO SCSI driver by default, which is what produces the **Code 28 (Drivers not installed)** error. I try the automated installer first, and fall back to the manual method only if that fails.

**Method A — guest tools installer (my default):** from inside the VM, run `virtio-win-gt-x64.msi` (or `virtio-win-guest-tools.exe`) off the mounted ISO. This installs every VirtIO driver at once, including SCSI.

**Method B — manual "Have Disk" fallback:** if the installer doesn't pick up the device, I force it manually in Device Manager: right-click the failing SCSI controller under *Other Devices* → **Update driver** → **Browse my computer for drivers** → **Let me pick from a list of available drivers on my computer** → **Show All Devices** → **Have Disk...** → browse to the VirtIO ISO's `vioscsi\w10\amd64\` folder (or `x86` for 32-bit Windows) → select `vioscsi.inf` → pick *Red Hat VirtIO SCSI pass-through controller* → finish.

## 4. Initialize and format the new drive

Once the driver shows as working, the drive still needs to be initialized inside Windows before it appears in File Explorer: **Disk Management** → accept the prompt to initialize as **GPT** → right-click the unallocated space → **New Simple Volume** → format as **NTFS** and assign a drive letter.
