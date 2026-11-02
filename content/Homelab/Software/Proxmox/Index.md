---
title: Proxmox Overview
description: Overview of the Proxmox virtualization platform, storage configuration, and management.
tags:
  - Homelab
  - Proxmox
  - Virtualization
  - Storage
---

## Overview

This page documents the Proxmox virtualization environment, including storage configuration and management practices.

---

## Proxmox Role and Purpose

Proxmox Virtual Environment (PVE) serves as the hypervisor platform for the home lab, running both LXC containers and KVM virtual machines. It provides centralized management of virtualization resources, backup operations, and storage administration.

---

## Storage Configuration

### Local Storage

The `local` storage pool combines what was previously separate `local` and `local-lvm` pools into a single ext4-based storage system. This configuration trades LVM-thin features (fast snapshots, thin provisioning) for simpler management and better performance for large VM disks.

- [[Homelab/Software/Proxmox/Proxmox Merge Local-LVM Into Local|Proxmox Merge Local-LVM Into Local]]

### Storage Trade-Offs

- Loss of LVM-thin features (fast snapshots, thin provisioning)
- VM disks become `.raw` or `.qcow2` files on ext4
- Containers on directory storage cannot use snapshots
- Backups and guest disks share one pool
- Requires retention limits or external backup targets

---

## Virtual Machine Management

Proxmox manages multiple VM types:

- **KVM VMs:** Full virtualization for Windows, Linux, and other operating systems
- **LXC Containers:** Lightweight containers for services and applications
- **GPU Passthrough:** Direct GPU access for VMs requiring hardware acceleration
```
