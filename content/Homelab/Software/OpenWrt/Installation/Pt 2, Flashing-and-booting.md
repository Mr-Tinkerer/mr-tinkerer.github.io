---
title: OpenWrt Installation Pt 2, Flashing and Booting
description: Getting the OpenWrt image onto a USB stick, discovering the .img file is the OS itself, and writing it to internal storage.
tags:
  - Homelab
  - OpenWrt
---

## Prerequisites

- [[Homelab/Software/OpenWrt/Installation/Pt 1, Choosing-the-router-os|Pt 1, Choosing the router OS]] — OpenWrt selected, `.img.gz` downloaded.

---

## Ventoy Attempt (Failed)

The extracted `.img` was first copied to an existing Ventoy USB stick. Booting produced an error that Ventoy's own docs explain: OpenWrt images don't ship the kernel modules Ventoy needs, so a separate `ventoy_openwrt.xz` plugin file has to be placed — unextracted — in a `ventoy` folder on the stick.

See [Ventoy boot error screenshot](Images/ventoy-openwrt-boot-error-missing-kernel-modules.png) for the Ventoy boot error caused by the missing kernel-module plugin.

Even after placing the plugin file (first extracted by mistake, then correctly left as the `.xz`), OpenWrt booted partway, printed kernel messages, then rebooted itself — the log showed `/dev/ventoy2: Can't lookup blockdev`. Ventoy was abandoned as the boot medium for this image.

## Direct Flash with Fedora Media Writer

The `.img` was instead flashed directly to a second USB stick using Fedora Media Writer. The stick didn't appear at all in the boot menu because the laptop's "BIOS" screen was actually a UEFI menu, and the downloaded image was the non-EFI variant. Re-flashing with the EFI image fixed this.

## Discovering .img is the OS, Not an Installer

Booting the EFI image reached a shell prompt (after learning that OpenWrt's install boot sequence pauses on "Please press Enter to activate this console"). At this point it became clear, via OpenWrt's x86 install guide, that the `.img` file already *is* the OS — flashing it to a USB stick installs OpenWrt *to that USB stick*, it does not produce an installer for internal storage.

### Live Environment Installation

To get OpenWrt onto the laptop's internal drive without removing it, the guide's live-environment method was used:

1. Boot a general-purpose Linux live environment (Arch Linux, already on hand)
2. Mount the USB stick holding the `.img`
3. `dd` the image onto the internal disk

After this, the laptop booted OpenWrt directly from internal storage with no USB media attached — the OS install itself was complete at this point.

---

Next: [[Homelab/Software/OpenWrt/Installation/Pt 3, Initial-access-and-root-password|Pt 3, Initial access and root password]]
