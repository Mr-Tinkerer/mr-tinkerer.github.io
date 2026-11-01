---
title: Gigabyte Gaming A16 Ga6h Overview
description: Daily-driver laptop hardware specifications and dual-GPU layout enabling GPU passthrough.
tags:
  - Hardware
  - Laptop
---

## Overview

This page documents the Gigabyte Gaming A16 (GA6H) laptop, configured as the daily driver running CachyOS with the [[Gigabyte-gaming-a16-ga6h-glossary#Niri|Niri]] window manager and Limine bootloader.

### Hardware Specifications

- **Dedicated GPU:** NVIDIA GeForce RTX 5060 Max-Q (Mobile)
  - Device ID: `10de:2d19`
  - GPU Audio: `10de:22eb`
- **Integrated GPU:** Intel graphics used for host display when the dedicated GPU is passed through to a VM

### Dual-GPU Architecture Benefits

The dual-GPU layout enables GPU passthrough to VMs while maintaining host display functionality. This architecture supports:

- Dedicated GPU passthrough for gaming VMs
- Host display on integrated GPU
- Seamless switching between host and VM GPU usage

See [[Windows-Gaming-VM/GPU-Passthrough/Pt 1, Creating-the-vm|GPU passthrough setup]] for implementation details.

## Firmware / BIOS Configuration

### IOMMU Configuration

The laptop's BIOS does not expose a dedicated "IOMMU" toggle. Instead:

- **Intel Virtualization Technology (VMX)** — Available in the Peripherals page
- **IOMMU verification** — Confirmed from within Linux via `sudo dmesg | grep -i IOMMU` (looking for "DMAR: IOMMU enabled")

See [BIOS screenshot](Images/bios-virtualization-technology-option.png) for the Peripherals page layout.

## Power Management

See [[Battery-management|Battery management]] for udev-driven automation adjusting power profile, brightness, and display profile on battery/plug events.
