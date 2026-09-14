---
title: Intel VMD
description: What Intel VMD is on this laptop's CPU, why it causes a Windows "no drives found" install error, and how I safely disabled it after installing Windows with it enabled.
tags:
  - Hardware
  - Windows
  - BIOS
---
# Intel VMD

## What is Intel VMD?

**Intel VMD (Volume Management Device)** is a hardware-based technology embedded directly inside my CPU (11th Gen and newer Intel chips have it).

Traditionally, NVMe SSDs connect directly to the CPU's PCIe lanes for maximum speed. However, this direct connection makes tasks like hot-swapping drives or handling drive failures risky, often leading to system crashes.

Intel VMD acts as a **hardware intermediary layer** between the CPU and the NVMe SSDs to provide enterprise-grade storage management.

### Key benefits

- **Isolated error handling:** intercepts drive glitches or failures so they don't crash the entire operating system.
- **Hot-plugging:** allows NVMe drives to be safely plugged in or removed while the system is running (crucial for servers).
- **Bootable NVMe RAID:** consolidates multiple NVMe drives under one controller so they can be configured in RAID arrays and booted from.

### The "no drives found" Windows gotcha

Because VMD acts as a middleman, standard Windows installation media often can't "see" past it. When installing Windows on a VMD-enabled system, I got a **"We couldn't find any drives"** error. This requires either loading the *Intel Rapid Storage Technology (IRST)* driver during setup or disabling VMD in the BIOS first.

---

## Converting an existing Windows 11 install (VMD enabled → disabled)

I had installed Windows 11 while Intel VMD was enabled on this laptop. Simply turning VMD off in the BIOS afterward caused a **Blue Screen of Death (BSOD)** on boot (`INACCESSIBLE_BOOT_DEVICE`). This happens because Windows keeps trying to use the VMD storage driver instead of the standard NVMe driver.

To safely disable VMD without losing data or reinstalling Windows, I used **Windows Safe Mode** to force Windows to swap drivers.

### Step-by-step procedure

1. **Force Safe Mode in Windows.** Open **Command Prompt as an Administrator** and run:
   ```cmd
   bcdedit /set {current} safeboot minimal
   ```
2. **Reboot and change BIOS settings.** Restart the PC and enter the BIOS (usually by tapping `F2` or `Del`). Find the **Intel VMD Technology** setting and change it to **Disabled**. Save and exit (`F10`).
3. **Turn off Safe Mode.** The PC boots into Safe Mode using generic NVMe drivers. Once at the desktop, open **Command Prompt as an Administrator** again and run:
   ```cmd
   bcdedit /deletevalue {current} safeboot
   ```
4. **Final reboot.** Restart the PC. It now boots normally with VMD completely disabled.

### What these commands actually do

Both commands use `bcdedit` (Boot Configuration Data Editor), the administrative tool that alters how the Windows boot manager behaves.

| Command | What it means | What it actually does |
|---|---|---|
| `bcdedit /set {current} safeboot minimal` | `/set` changes a setting; `{current}` targets the active OS; `safeboot minimal` forces Safe Mode | Tells Windows to ignore third-party drivers (like VMD) on the next boot and use bare-minimum generic storage drivers instead — this prevents the BSOD when VMD is turned off |
| `bcdedit /deletevalue {current} safeboot` | `/deletevalue` erases a rule; `{current}` targets the active OS; `safeboot` is the Safe Mode flag | Deletes the Safe Mode restriction *after* Windows has adapted to the standard NVMe connection, allowing the computer to boot normally on subsequent restarts |
