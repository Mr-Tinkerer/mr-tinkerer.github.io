---
title: "Pt 3, Isolating the GPU"
description: Binding the dedicated GPU to VFIO at boot so the host never claims it.
tags:
  - Virtual_Machines
  - VFIO
---
# Pt 3, Isolating the GPU

# Prerequisites
- [[Pt 1, Creating-the-vm|Pt 1, Creating the VM]]
- [[Pt 2, Enabling-iommu|Pt 2, Enabling IOMMU]]

---

## Goal

Give the dedicated GPU and its audio device to the [[Windows-gaming-vm-glossary#VFIO|VFIO]] driver during boot, before NVIDIA's own driver can claim them — this makes the GPU unusable by the host until VFIO releases it.

## Get the PCI IDs

The IOMMU-group listing from [[Pt 2, Enabling-iommu|Pt 2]] already lists each device's PCI ID in `[vendor:device]` form — for this laptop's GPU, `10de:2d19` (GPU) and `10de:22eb` (audio).

## Tell the kernel to use VFIO for those IDs

Create `/etc/modprobe.d/vfio.conf`:

```
options vfio-pci ids=10de:2d19,10de:22eb
```

## Load VFIO before the NVIDIA driver in the initramfs

Which initramfs generator is in use can be checked with:

```
which dracut mkinitcpio update-initramfs mkinitrd booster
```

CachyOS (and vanilla Arch) uses [[Windows-gaming-vm-glossary#mkinitcpio|mkinitcpio]]. In `/etc/mkinitcpio.conf`:

- Add `vfio_pci vfio vfio_iommu_type1` to the `MODULES=()` line — if `nvidia` is also listed there, make sure it comes *after* the VFIO modules.
- Make sure `modconf` is present in `HOOKS=()`.

Rebuild the initramfs. On CachyOS with Limine this is `limine-mkinitcpio -P` (not the `mkinitcpio -P` used with other bootloaders).

Reboot. If only one monitor is available and it's plugged into the dedicated GPU, expect a black screen — Linux is only rendering through the integrated GPU now. Switch the monitor cable to the IGPU if there's no second monitor.

## A boot-breaking dead end

I also tried the opposite of the module ordering above: loading the NVIDIA driver at boot *and* keeping VFIO's claim, by inserting `nvidia nvidia_modeset nvidia_uvm nvidia_drm` ahead of the VFIO modules in `mkinitcpio.conf`, to see if I could have both available from boot without a later rebind step. This broke CachyOS's boot process entirely, requiring an [Arch chroot](https://wiki.archlinux.org/title/Chroot) rescue to revert:

![[cachyos-boot-failure-nvidia-module-order.png]]
*CachyOS failing to boot after reordering NVIDIA modules ahead of VFIO in mkinitcpio.conf — a real failure mode worth knowing about before touching that ordering.*

The working alternative is the approach documented above and in [[Pt 6, Dynamic-driver-switching|Pt 6]]: keep VFIO claiming the GPU at boot (as configured above), and do the actual host/guest switch live at runtime via the `driver_override` dance, rather than trying to make both drivers available from boot. I confirmed this with `lspci -nnk`, showing both the GPU and its audio device successfully carrying the `vfio-pci` driver after a manual switch with no boot-time interference:

![[gpu-audio-bound-to-vfio-pci-verified.png]]
*`lspci -nnk` confirming both the GPU and its audio device are bound to vfio-pci — the state the automation scripts drive the devices into before the VM starts.*

## Verifying it worked

```
lspci -nnk -d 10de:2d19
```

```
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GB206M [GeForce RTX 5060 Max-Q / Mobile] [10de:2d19] (rev a1)
	Subsystem: Gigabyte Technology Co., Ltd Device [1458:6019]
	Kernel driver in use: vfio-pci
	Kernel modules: nouveau, nvidia_drm, nvidia
```

`Kernel driver in use: vfio-pci` confirms the GPU is isolated. Repeat for the audio device's PCI ID. Note the numeric PCI addresses shown at the start of the output (e.g. `01:00.0`) — they're needed for [[Pt 4, Attaching-the-gpu|Pt 4]].

---

Next: [[Pt 4, Attaching-the-gpu|Pt 4, Attaching the GPU to the VM]].
