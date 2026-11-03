---
title: Troubleshooting a Windows 10 VirtIO SCSI Drive
description: Fixing a secondary SCSI drive that Windows 10 wouldn't detect, or flagged with a Code 28/Code 10 error, under libvirt/KVM.
tags:
  - Virtual_Machines
  - QEMU
  - Windows
  - Troubleshooting
---

## Overview

When a secondary SCSI drive was added to a Windows 10 VM, it wasn't detected, showed a yellow exclamation mark (Code 28/Code 10), or rejected the VirtIO drivers. This page documents the resolution steps.

## Step 1: Verify the SCSI Controller Model

Windows throws a **Code 10 (Device cannot start)** error if the underlying emulated controller is misconfigured.

### Configuration Steps

1. Shut down the VM
2. In Virt Manager, open the VM's details and select the **SCSI Controller** in the hardware panel
3. Change the **Model** dropdown from `LSI Logic` (the default) to **VirtIO SCSI**
4. Click **Apply** and start the VM

Via `virsh edit`, the controller XML should read:

```xml
<controller type='scsi' index='0' model='virtio-scsi'/>
```

## Step 2: Get the VirtIO Drivers ISO

The driver ISO (e.g., `virtio-win-0.1.285-1.iso`) was downloaded from the Fedora VirtIO archive, then added as a CD-ROM device (or an existing one was pointed at the downloaded .iso) and applied.

## Step 3: Install the VirtIO SCSI Driver

Windows 10 doesn't include the VirtIO SCSI driver by default, which produces the **Code 28 (Drivers not installed)** error. The automated installer was tried first, with the manual method used as a fallback if that fails.

### Method A — Guest Tools Installer (Default)

From inside the VM, `virtio-win-gt-x64.msi` (or `virtio-win-guest-tools.exe`) was run off the mounted ISO. This installs every VirtIO driver at once, including SCSI.

### Method B — Manual "Have Disk" Fallback

If the installer didn't pick up the device, the driver was forced manually in Device Manager:

1. Right-click the failing SCSI controller under *Other Devices*
2. **Update driver**
3. **Browse my computer for drivers**
4. **Let me pick from a list of available drivers on my computer**
5. **Show All Devices**
6. **Have Disk...**
7. Browse to the VirtIO ISO's `vioscsi\\w10\\amd64\\` folder (or `x86` for 32-bit Windows)
8. Select `vioscsi.inf`
9. Pick *Red Hat VirtIO SCSI pass-through controller*
10. Finish

## Step 4: Initialize and Format the New Drive

Once the driver shows as working, the drive still needs to be initialized inside Windows before it appears in File Explorer:

1. **Disk Management**
2. Accept the prompt to initialize as **GPT**
3. Right-click the unallocated space
4. **New Simple Volume**
5. Format as **NTFS** and assign a drive letter
