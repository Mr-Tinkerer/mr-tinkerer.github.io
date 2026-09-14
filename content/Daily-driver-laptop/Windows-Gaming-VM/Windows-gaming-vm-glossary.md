---
title: Windows Gaming VM Glossary
description: Terms used across the Windows Gaming VM / GPU passthrough bucket.
tags:
  - Glossary
---
# Glossary

## Hypervisor (Type 1 vs Type 2)
A Type 2 hypervisor runs on top of an existing OS and must compete with it for hardware access. A Type 1 hypervisor runs directly on the physical hardware (or, in QEMU/KVM's case, gets direct hardware access via the kernel), giving VMs near-native performance and making full device passthrough practical. See [AWS: Type 1 vs Type 2 hypervisors](https://aws.amazon.com/compare/the-difference-between-type-1-and-type-2-hypervisors/).

## QEMU
A machine emulator/virtualizer that can run VMs on top of a Linux distro, emulating CPU, storage, network, and other virtual hardware for the guest. On its own QEMU is a Type 2-style emulator; paired with [[Windows-gaming-vm-glossary#KVM|KVM]] it gets direct hardware access through the kernel, which is what makes GPU passthrough to the Windows VM documented in this bucket practical. See [qemu.org](https://www.qemu.org/) / [Wikipedia: QEMU](https://en.wikipedia.org/wiki/QEMU).

## KVM
Kernel-based Virtual Machine — a virtualization module built into the Linux kernel that lets QEMU bypass the host OS and access hardware directly, turning QEMU into a de facto Type 1 hypervisor. See [Wikipedia: KVM](https://en.wikipedia.org/wiki/Kernel-based_Virtual_Machine).

## libvirt
A toolkit that manages QEMU/KVM VMs and stores their configuration as XML instead of long command lines. I interact with it mostly through [[Windows-gaming-vm-glossary#Virt Manager|Virt Manager]]'s GUI, but drop into raw XML editing or `virsh` directly for things the GUI doesn't expose (IVSHMEM devices, CPU pinning, libvirt hooks). It also runs its own DHCP/DNS/NAT stack for each virtual network it creates — see [[Libvirt-networking-and-firewall]] for how that interacts with the host firewall. See [libvirt.org](https://libvirt.org/).

## Virt Manager
A GUI front-end for libvirt (and by extension QEMU/KVM) that's used throughout this bucket's setup — creating the VM, fixing its CPU topology, attaching PCI devices for GPU passthrough, and editing the VM's underlying XML directly (e.g. to add the IVSHMEM device for Looking Glass) when the GUI alone isn't enough. See [virt-manager.org](https://virt-manager.org/).

## IOMMU
Input-Output Memory Management Unit — hardware that maps devices efficiently to system RAM, required for a VM to access physical hardware (like a GPU) directly. See [Arch Wiki: PCI passthrough via OVMF](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Prerequisites).

## VFIO
Virtual Function I/O — the Linux kernel driver framework used to hand a PCI device to a VM while preventing the host from using it. See [Arch Wiki: PCI passthrough via OVMF](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Isolating_the_GPU).

## IOMMU group
The set of PCI devices that IOMMU isolation groups together; every device in a group must be passed through together, since the IOMMU can't isolate memory access between devices sharing a group. On my laptop the GPU and its audio device share one clean group with nothing else in it — see [[GPU-Passthrough/Pt 2, Enabling-iommu|Pt 2]] for how I checked this and why a "dirty" group (one containing unrelated devices) would have forced passing those through too. See [Arch Wiki: Ensuring that the groups are valid](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Ensuring_that_the_groups_are_valid).

## VirtIO drivers
A set of paravirtualized Windows drivers (disk, network, memory ballooning) that let a Windows guest talk to QEMU's virtual devices efficiently. Maintained by the Fedora Project. See [Proxmox: Windows VirtIO Drivers](https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers).

## QEMU Guest Agent
A guest-side daemon that exchanges information with the host — clean shutdowns, filesystem freeze for snapshots/backups, and automatic guest display resizing. I install it alongside the [[Windows-gaming-vm-glossary#VirtIO drivers|VirtIO drivers]] right after Windows setup (see [[GPU-Passthrough/Pt 1, Creating-the-vm|Pt 1]]) since libvirt/Virt Manager can't cleanly shut the VM down or resize its display without it running inside the guest. See [Proxmox: qemu-guest-agent](https://pve.proxmox.com/wiki/Qemu-guest-agent#Windows).

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

## SPICE
A remote-display protocol QEMU can expose a VM's console over, viewed with `virt-viewer`/`remote-viewer` or through Virt Manager's built-in console. It's the display path I use before/instead of [[Windows-gaming-vm-glossary#Looking Glass|Looking Glass]] — see [[Spice-auto-resize-fix]] for a Wayland-specific scaling bug I hit with it. See [spice-space.org](https://www.spice-space.org/).

## nftables
The Linux kernel's current packet-filtering/NAT framework (successor to iptables), organized into tables and chains hooked into points like `input`/`forward`/`output`. libvirt, UFW, and any manual rules I write all install their own independent tables into the same nftables framework — see [[Libvirt-networking-and-firewall]] for why that independence matters. See [Arch Wiki: nftables](https://wiki.archlinux.org/title/Nftables).

## UFW
"Uncomplicated Firewall" — a policy front-end over nftables/iptables-nft. It manages its own tables and chains independently of anything I write by hand in `/etc/nftables.conf`, which is the root cause of a libvirt DHCP/DNS failure documented in [[Libvirt-networking-and-firewall]]. See [Ubuntu: UFW](https://help.ubuntu.com/community/UFW).

## dnsmasq
A lightweight DHCP/DNS server; libvirt runs one instance per virtual network, bound to that network's bridge interface, to hand out leases and resolve names for VMs on it. See [dnsmasq project page](https://thekelleys.org.uk/dnsmasq/doc.html).
