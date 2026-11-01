---
title: OpenWrt Installation Pt 1, Choosing the Router OS
description: Why OpenWrt was picked over pfSense and OPNsense, and which install image to download.
tags:
  - Homelab
  - OpenWrt
---

## Prerequisites

None — this is the first step.

## Why Not pfSense or OPNsense

Three DIY router OSes were considered: pfSense, OPNsense, and OpenWrt.

### pfSense

Ruled out because the installer requires an internet connection, which isn't available before the router itself is configured to reach the building's Wi-Fi — a chicken-and-egg problem in this specific environment.

### OPNsense

Ruled out because the FreeBSD base has inconsistent Wi-Fi card driver support, and even where a card is supported, driver feature/performance parity with Linux is often worse.

### OpenWrt

Chosen because:

- Linux-based — Wi-Fi driver support tracks the Linux kernel
- Runs on x86 hardware despite being designed for embedded routers
- The only option not already familiar from prior projects

## Picking a Download Image

From OpenWrt's download page, the `x86/64` target was selected. Within that target, images are offered as `ext4` or squashfs, each as `combined` or `rootfs`, and as EFI or non-EFI:

### Filesystem Choice

**ext4** was chosen over squashfs because squashfs is read-only, and installing Wi-Fi drivers after the fact requires writing to the root filesystem.

### Image Type

**combined** was chosen over `rootfs` on the assumption it includes more of what's needed (bootloader files, etc).

### EFI Boot

The **non-EFI** image was tried first since the target motherboard's UEFI boot support was uncertain. This assumption turned out to be wrong — see Pt 2.

---

Next: [[Homelab/Software/OpenWrt/Installation/Pt 2, Flashing and booting|Pt 2, Flashing and booting]]
