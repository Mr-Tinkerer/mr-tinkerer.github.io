---
title: Windows Gaming VM Glossary
description: Terms used across the Windows Gaming VM / GPU passthrough bucket.
tags:
  - Glossary
---
# Glossary

## Hypervisor (Type 1 vs Type 2)
A Type 2 hypervisor runs on top of an existing OS and must compete with it for hardware access. A Type 1 hypervisor runs directly on the physical hardware (or, in QEMU/KVM's case, gets direct hardware access via the kernel), giving VMs near-native performance and making full device passthrough practical. See [AWS: Type 1 vs Type 2 hypervisors](https://aws.amazon.com/compare/the-difference-between-type-1-and-type-2-hypervisors/).

## IOMMU
Input-Output Memory Management Unit — hardware that maps devices efficiently to system RAM, required for a VM to access physical hardware (like a GPU) directly. See [Arch Wiki: PCI passthrough via OVMF](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Prerequisites).

## VFIO
Virtual Function I/O — the Linux kernel driver framework used to hand a PCI device to a VM while preventing the host from using it. See [Arch Wiki: PCI passthrough via OVMF](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Isolating_the_GPU).

## IOMMU group
The set of PCI devices that IOMMU isolation groups together; every device in a group must be passed through together, since the IOMMU can't isolate memory access between devices sharing a group. On my laptop the GPU and its audio device share one clean group with nothing else in it — see [[GPU-Passthrough/Pt 2, Enabling-iommu|Pt 2]] for how I checked this and why a "dirty" group (one containing unrelated devices) would have forced passing those through too. See [Arch Wiki: Ensuring that the groups are valid](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Ensuring_that_the_groups_are_valid).

## Looking Glass
A tool that reads a passthrough GPU's raw frames to display and control the VM with low latency, instead of relying on a physical monitor plugged into the GPU. See [looking-glass.io](https://looking-glass.io/).

## IVSHMEM
Inter-VM Shared Memory — a device that shares a memory region between host and guest; Looking Glass uses it to move frame data. See [QEMU: ivshmem spec](https://www.qemu.org/docs/master/specs/ivshmem-spec.html).

## mkinitcpio
Arch/CachyOS's initramfs generator; the `MODULES=()` and `HOOKS=()` lines in `/etc/mkinitcpio.conf` control which kernel modules (e.g. `vfio_pci`) load before others (e.g. `nvidia`) during boot. See [Arch Wiki: PCI passthrough via OVMF — mkinitcpio](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#mkinitcpio).

## Libvirt hook
A script at `/etc/libvirt/hooks/qemu` that libvirt executes automatically on VM lifecycle events (e.g. `prepare`/`begin`, `release`/`end`), used here to run the GPU passthrough scripts automatically on VM start/stop. See [libvirt: hooks for specific system management](https://www.libvirt.org/hooks.html).

## WirePlumber
The session/policy manager for [PipeWire](https://www.pipewire.org/) (the Linux audio server). PipeWire keeps a device open as long as WirePlumber considers it active, which gets in the way of GPU passthrough: the GPU's HDMI audio device won't cleanly unbind from the host while WirePlumber still holds it, so I disable it via a WirePlumber rule before the driver switch and re-enable it afterward — see [[GPU-Passthrough/Pt 6, Dynamic-driver-switching|Pt 6]]. See [WirePlumber docs](https://pipewire.pages.freedesktop.org/wireplumber/).

