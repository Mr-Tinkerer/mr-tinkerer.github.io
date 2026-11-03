---
title: Lid and Power Management
description: Keeping the server running with the lid closed, and why the USB Wi-Fi dongle forces the lid to stay open on this laptop.
tags:
  - Asus-X555q
  - Hardware
  - Power
---

## Overview

This page documents lid and power management issues with the Asus X555q.

## Preventing Suspend on Lid Close

Debian/Proxmox uses systemd, so lid behavior is controlled in `logind.conf`. Setting `HandleLidSwitch`, `HandleLidSwitchDocked`, and `HandleLidSwitchExternalPower` all to `ignore`, then restarting the service (`systemctl restart systemd-logind`), stops the machine from suspending when the lid closes.

## Why the Lid Still Has to Stay Open

Closing the lid on this laptop cuts power to its USB ports — and the Wi-Fi connection runs through a USB dongle, not the (faulty) internal card. With the lid fully closed, the dongle loses power and the machine drops off the network entirely.

This is a hardware/firmware behavior, not something `logind` or any OS setting can override — the BIOS on this model has no option to keep USB power on with the lid closed, and the BIOS is barebones with no likely fix even after an update. Updating the BIOS was also ruled out as too risky given how unreliable power was at the time.

The workaround is physical: an object is wedged in the hinge to keep the lid open at roughly 45 degrees, which is enough to keep the USB ports powered while still letting the laptop sit flat.

## Gotchas

- If a USB-attached peripheral (Wi-Fi dongle, external drive, etc.) drops out specifically on lid-close, suspect the lid switch cutting USB port power at the hardware/BIOS level rather than an OS power-management bug
