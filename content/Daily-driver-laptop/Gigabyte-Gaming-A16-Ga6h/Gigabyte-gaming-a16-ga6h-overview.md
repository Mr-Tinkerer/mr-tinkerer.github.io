---
title: Gigabyte Gaming A16 Ga6h Overview
description: The physical daily-driver laptop — a Gigabyte Gaming A16 (GA6H) running CachyOS with Niri.
tags:
  - Hardware
  - Laptop
---
# Overview

This is my daily-driver laptop: a [Gigabyte Gaming A16 (GA6H)](https://www.gigabyte.com/Laptop/GIGABYTE-GAMING-A16-GA6H), running [CachyOS](https://cachyos.org/) with the [[Gigabyte-gaming-a16-ga6h-glossary#Niri|Niri]] window manager and [Limine](https://github.com/Limine-Bootloader/Limine) as the bootloader.

It has a dual-GPU layout typical of gaming laptops:
- An NVIDIA GeForce RTX 5060 Max-Q (Mobile) dedicated GPU (`10de:2d19`, plus its audio device `10de:22eb`).
- An Intel integrated GPU I use for the host display when the dedicated GPU is passed through to a VM.

This dual-GPU layout is what makes [[Windows-Gaming-VM/GPU-Passthrough/Pt 1, Creating-the-vm|GPU passthrough]] possible for me without losing the ability to use the host OS.

---

## Firmware / BIOS notes

This laptop's BIOS doesn't expose a separate "IOMMU" toggle. Only [Intel Virtualization Technology (VMX)](Images/bios-virtualization-technology-option.png) is present as a peripherals-page option; I confirm IOMMU support instead from within Linux via `sudo dmesg | grep -i IOMMU` (looking for `DMAR: IOMMU enabled`).

![[bios-virtualization-technology-option.png]]
*The BIOS's Peripherals page — there is no dedicated IOMMU/VT-d toggle, only Intel (VMX) Virtualization Technology. This is a hardware/firmware constraint specific to this laptop's BIOS layout.*

---

## On-battery behavior

See [[Battery-management]] for the udev-driven automation I use to adjust power profile, brightness, and display profile when I plug/unplug the laptop.
