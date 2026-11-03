---
title: Proxmox Virtual Machines on Proxmox Hypervisor
description: Attaching install media and creating the home lab's VMs in Proxmox.
tags:
  - Homelab
  - Proxmox
---

## Overview

This page covers attaching install media and creating the home lab's VMs in Proxmox.

## Attaching Install Media

OS ISOs were downloaded and uploaded to Proxmox's local storage, then attached to new VMs as install media. This follows the standard Proxmox storage documentation for adding an ISO-capable storage location.

## Creating the VMs

Most VMs were created with the same baseline settings (CPU, RAM, and disk sized per-role), one at a time through the Proxmox web UI's VM creation wizard: general info, OS/ISO, system settings, disk, CPU, memory, network, then confirm.

### Windows Server Exception

The Windows Server 2022 VM was given 4GB RAM and 32GB storage instead of the shared baseline, to match Windows's higher minimum requirements.

## Related Documentation

- [[Homelab/Logical/Virtual-machines/Vm-allocation|VM allocation]] — The resulting roster of VMs, what each one runs, and why
- [[Libvirt-glossary/Virtual-disks-reference|Virtual disks reference]] — QEMU/libvirt virtual disk concepts (image formats, thin-provisioning, disk expansion) that apply here too, since Proxmox VMs are QEMU-backed under the hood
