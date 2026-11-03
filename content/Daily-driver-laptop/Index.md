---
title: Daily Driver Laptop
description: The user's actual daily-driver laptop — a Gigabyte Gaming A16 running CachyOS/Niri — and the software and automation built around it.
tags:
  - Laptop
---

# Daily Driver Laptop

This domain covers the daily-driver laptop (not the separate failed/broken Asus laptop documented elsewhere in this vault): its hardware, the Windows gaming VM set up on it with GPU passthrough, and how backup storage is consumed from the home NAS on it.

At this point, there aren't enough Hardware entries (only one) alongside Software/Logical entries to trigger the Hardware/Software/Logical bucket split (§3) — buckets stay flat at the domain root.

# Gigabyte-Gaming-A16-Ga6h

The physical laptop itself: its dual-GPU layout, BIOS/firmware quirks relevant to virtualization, the udev-driven automatic battery/power management running on it, and the input, theme, and storage-controller (Intel VMD) fixes it needed.

- [[Gigabyte-Gaming-A16-Ga6h/Gigabyte-gaming-a16-ga6h-glossary|Glossary]]
- [[Gigabyte-Gaming-A16-Ga6h/Gigabyte-gaming-a16-ga6h-overview|Overview]]
- [[Gigabyte-Gaming-A16-Ga6h/Battery-management|Battery management]]
- [[Gigabyte-Gaming-A16-Ga6h/Disable-while-typing-on-wayland|Disable-while-typing on Wayland]]
- [[Gigabyte-Gaming-A16-Ga6h/Intel-vmd|Intel VMD]]
- [[Gigabyte-Gaming-A16-Ga6h/Niri-dark-theme-fix|Niri dark theme fix]]

# Windows-Gaming-VM

A Windows 10 VM with GPU passthrough (VFIO), set up so playing games or running untrusted software can use the dedicated GPU at full performance while keeping the host usable, with automatic driver switching wired into VM start/stop.

- [[Windows-Gaming-VM/Windows-gaming-vm-glossary|Glossary]]
- GPU-Passthrough (sequential setup):
  - [[Windows-Gaming-VM/GPU-Passthrough/Pt 1, Creating-the-vm|Pt 1, Creating the VM]]
  - [[Windows-Gaming-VM/GPU-Passthrough/Pt 2, Enabling-iommu|Pt 2, Enabling IOMMU]]
  - [[Windows-Gaming-VM/GPU-Passthrough/Pt 3, Isolating-the-gpu|Pt 3, Isolating the GPU]]
  - [[Windows-Gaming-VM/GPU-Passthrough/Pt 4, Attaching-the-gpu|Pt 4, Attaching the GPU to the VM]]
  - [[Windows-Gaming-VM/GPU-Passthrough/Pt 5, Setting-up-looking-glass|Pt 5, Setting up Looking Glass]]
  - [[Windows-Gaming-VM/GPU-Passthrough/Pt 6, Dynamic-driver-switching|Pt 6, Dynamic driver switching]]
  - [[Windows-Gaming-VM/GPU-Passthrough/Pt 7, Automating-with-libvirt-hooks|Pt 7, Automating with libvirt hooks]]
- [[Windows-Gaming-VM/Post-install-tweaks|Post-install tweaks]]



# NAS-Backup

How this laptop consumes storage from the home NAS: mounting its share on demand and automatically backing up Documents to it.

- [[NAS-Backup/Nas-backup-glossary|Glossary]]
- [[NAS-Backup/Automated-document-backup|Automated document backup]]
- [[Daily-driver-laptop/Software/Hermes/Hermes Agent Desktop Installation on CachyOS|Hermes]] — Hermes Agent desktop installation on CachyOS. (Glossary)
