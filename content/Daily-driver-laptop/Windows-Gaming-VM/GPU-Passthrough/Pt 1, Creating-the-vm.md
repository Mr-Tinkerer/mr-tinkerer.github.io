---
title: "Pt 1, Creating the VM"
description: Creating the base Windows 10 VM used for GPU passthrough gaming.
tags:
  - Virtual_Machines
  - QEMU
---
# Pt 1, Creating the VM

# Prerequisites
None — this is the first step.

---

## Goal

Build a Windows 10 VM under [[Windows-gaming-vm-glossary#QEMU|QEMU]]/[[Windows-gaming-vm-glossary#KVM|KVM]] (managed with [[Windows-gaming-vm-glossary#libvirt|libvirt]] and [[Windows-gaming-vm-glossary#Virt Manager|Virt Manager]]) that can later be given the laptop's dedicated GPU. Reasons for this setup: confirming whether a game issue comes from [Proton](https://github.com/ValveSoftware/Proton) rather than the game itself, and running untrusted software isolated from the host while still getting full GPU performance.

I avoided Type 2 hypervisors (VirtualBox, VMware Workstation) because they compete with the host OS for hardware access, making full GPU passthrough impractical — see [[Windows-gaming-vm-glossary#Hypervisor (Type 1 vs Type 2)|Hypervisor]]. QEMU with KVM behaves like a Type 1 hypervisor since KVM lets it access hardware directly through the kernel.

## Prerequisites for GPU passthrough in general

- Two GPUs: one dedicated GPU to hand to the VM, and a second (an integrated GPU is fine) so the host OS still has something to render with once the dedicated GPU is detached.
- A CPU/motherboard with IOMMU support (Intel VT-d/VT-x, or AMD-Vi/AMD chips from the Bulldozer generation, October 2011, onward).
- A way to see the VM's video output — either a second monitor (one on each GPU) or a way to swap a single monitor's cable between them.
- Any modern Linux distro (the tooling used here doesn't support Windows or macOS as the host).

I did this on the [[Gigabyte-gaming-a16-ga6h-overview|Gigabyte Gaming A16 Ga6h]] laptop running CachyOS.

## Getting the ISO and picking a Windows version

I deliberately used Windows 10 Home over Windows 11: it avoids Windows 11's stricter install requirements and mandatory Microsoft-account sign-in, and since this VM will almost never touch the network, Windows 10's eventual end of support isn't a real concern to me. Microsoft still hosts the [Windows 10 ISO](https://www.microsoft.com/en-ca/software-download/windows10iso) directly.

## VM configuration

- 16 GB RAM, 6 CPU cores, 64 GB disk (Windows 10 plus drivers doesn't need much more).
- On the final creation screen, select **Customize configuration before install**.
- Fix the CPU topology: by default Virt Manager gives the VM multiple CPU *sockets* rather than multiple *cores* on one socket — Windows performs much worse under this default and needs the topology corrected manually before install.
- Disable the VM's virtual NIC device in Virt Manager/libvirt (the VM's hardware definition, not a setting inside Windows itself) during install, so Windows 10 setup can be done fully offline (skip the Microsoft account requirement).

## Installing Windows 10

Standard Windows 10 install: no license key, Windows 10 Home edition, use the entire (virtual) drive. During the offline setup screens, choose **I don't have internet** to skip network/Microsoft-account setup.

When creating the local account, I'm deliberate about the username/password used — a weak placeholder credential for a VM that stays offline is a different risk profile than the same credential on a networked machine, but it's still worth choosing consciously rather than by habit. I used a deliberate placeholder credential just to get past the setup screen, and changed it immediately afterward.

## Drivers and updates

After install, re-enable that virtual NIC device in Virt Manager/libvirt and install the [[Windows-gaming-vm-glossary#VirtIO drivers|VirtIO drivers]] and [[Windows-gaming-vm-glossary#QEMU Guest Agent|QEMU Guest Agent]] from the [Fedora-maintained VirtIO driver repo](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.285-1/) — run `virtio-win-guest-tools.exe` with default options. Remove the ISO from the virtual CD drive afterward.

Let Windows install any pending updates, then take a VM snapshot of this clean, updated state before moving on to GPU passthrough setup.

---

Next: [[Pt 2, Enabling-iommu|Pt 2, Enabling IOMMU]].
