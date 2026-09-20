---
title: "Pt 4, Attaching the GPU to the VM"
description: Adding the isolated PCI devices to the VM in Virt Manager.
tags:
  - Virtual_Machines
  - VFIO
---
# Pt 4, Attaching the GPU to the VM

# Prerequisites
- [[Pt 1, Creating-the-vm|Pt 1, Creating the VM]]
- [[Pt 2, Enabling-iommu|Pt 2, Enabling IOMMU]]
- [[Pt 3, Isolating-the-gpu|Pt 3, Isolating the GPU]]

---

## Adding the PCI devices

In [[Libvirt-glossary#Virt Manager|Virt Manager]], open the VM's Hardware page, click **Add Hardware**, then select **PCI Host Device**. Pick the entries matching the GPU's PCI numbers noted in [[Pt 3, Isolating-the-gpu|Pt 3]] (this matters if the device name alone is ambiguous). Add every device from the same [[Libvirt-glossary#IOMMU group|IOMMU group]] — for this laptop, both the GPU (`01:00.0`) and its audio device (`01:00.1`).

## First boot with the GPU attached

Start the VM. If the GPU is plugged into a monitor, it may still show nothing at first — this is expected, since the GPU driver isn't installed in the guest yet. Install the NVIDIA driver inside the VM; the monitor connected to the GPU then starts displaying output.

![[nvidia-driver-install-finished-in-vm.jpg]]
*The NVIDIA driver installer finishing inside the VM, with the guest desktop already rendering — confirms the passed-through GPU is being driven correctly by the guest.*

---

Next: [[Pt 5, Setting-up-looking-glass|Pt 5, Setting up Looking Glass]].
