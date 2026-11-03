---
title: Fedora VM Storage Expansion glossary
description: Glossary of terms related to LVM and XFS storage management in Fedora Linux VMs.
tags:
  - Homelab
  - Proxmox
  - Fedora
  - LVM
  - XFS
  - glossary
---

## Overview

This glossary defines terms used in the Fedora VM storage expansion documentation.

---

## LVM (Logical Volume Manager)

A Linux storage abstraction that allows dynamic management of physical storage into logical volumes. LVM provides features like volume resizing, snapshot creation, and thin provisioning.

### Physical Volume (PV)

A partition or entire disk that has been initialized for use with LVM. The physical volume is the building block of LVM storage.

### Volume Group (VG)

A collection of one or more physical volumes that can be treated as a single storage pool. Logical volumes are created from space within a volume group.

### Logical Volume (LV)

A partition-like container created from space within a volume group. Logical volumes can be resized dynamically without rebooting the system.

## XFS

A journaling filesystem developed by SUSE and Red Hat. XFS is commonly used as the default filesystem for Fedora and supports online resizing with `xfs_growfs`.

### xfs_growfs

A command used to expand an XFS filesystem to use all available space on its underlying logical volume. The filesystem must be mounted for this operation.

## growpart

A utility from the `cloud-utils-growpart` package used to expand Linux partitions into newly available disk space. Essential for LVM-based storage expansion workflows.

## pvresize

A command that tells LVM a physical volume has grown and should be updated to reflect the new size. Used after expanding a partition with `growpart`.

## lvextend

A command that increases the size of a logical volume by allocating free space from its volume group. The `-l +100%FREE` flag allocates all available free space.

## findmnt

A command that displays mount point information for filesystems. Used to verify which device is mounted at a specific mount point.

## df

A command that reports disk space usage on mounted filesystems. The `-h` flag displays sizes in human-readable format.
