---
title: Virtual machines
description: Attaching install media and creating the home lab's VMs in Proxmox.
tags:
  - Homelab
  - Proxmox
---
# Attaching install media

I downloaded the OS ISOs I needed and uploaded them to Proxmox's local storage, then attached them to new VMs as install media, following the standard [Proxmox storage documentation](https://pve.proxmox.com/pve-docs/chapter-pvesm.html) for adding an ISO-capable storage location.

# Creating the VMs

I created most VMs with the same baseline settings (CPU, RAM, and disk sized per-role), one at a time through the Proxmox web UI's VM creation wizard: general info, OS/ISO, system settings, disk, CPU, memory, network, then confirm.

The one exception was the Windows Server 2022 VM, which I gave 4GB RAM and 32GB storage instead of the shared baseline, to match Windows's higher minimum requirements.

See [[Homelab/Logical/Virtual-machines/Vm-allocation|VM allocation]] for the resulting roster of VMs, what each one runs, and why. See [[Libvirt-glossary/Virtual-disks-reference|Virtual disks reference]] for QEMU/libvirt virtual disk concepts (image formats, thin-provisioning, disk expansion) that apply here too, since Proxmox VMs are QEMU-backed under the hood.
