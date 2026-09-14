---
title: "Pt 1: Choosing the router OS"
description: Why OpenWrt was picked over pfSense and OPNsense, and which install image to download.
tags:
  - Homelab
  - OpenWrt
---
# Prerequisites

None — this is the first step.

---
# Why not pfSense or OPNsense

I considered three DIY router OSes: [[Homelab/Software/OpenWrt/Openwrt-glossary#pfSense|pfSense]], [[Homelab/Software/OpenWrt/Openwrt-glossary#OPNsense|OPNsense]], and [[Homelab/Software/OpenWrt/Openwrt-glossary#OpenWrt|OpenWrt]].

- I ruled out **pfSense** because its installer requires an internet connection, which isn't available before the router itself is configured to reach the building's Wi-Fi — a chicken-and-egg problem in this specific environment.
- I ruled out **OPNsense** because its FreeBSD base has inconsistent Wi-Fi card driver support, and even where a card is supported, driver feature/performance parity with Linux is often worse.
- I chose **OpenWrt**: it's Linux-based (so Wi-Fi driver support tracks the Linux kernel), it can run on x86 hardware despite being designed for embedded routers, and it was the only one of the three I wasn't already familiar with from prior projects.

# Picking a download image

From [OpenWrt's download page](https://downloads.openwrt.org/), I selected the `x86/64` target. Within that target, images are offered as `ext4` or [[Homelab/Software/OpenWrt/Openwrt-glossary#SquashFS|squashfs]], each as `combined` or `rootfs`, and as EFI or non-EFI:

- I chose **ext4** over squashfs because squashfs is read-only, and installing Wi-Fi drivers after the fact requires writing to the root filesystem.
- I chose **combined** over **rootfs** on the assumption it includes more of what's needed (bootloader files, etc).
- I tried the **non-EFI** image first, since I wasn't sure whether the target motherboard supported UEFI boot. (That assumption turned out to be wrong — see [[Homelab/Software/OpenWrt/Installation/Pt 2, Flashing-and-booting|Pt 2]].)

---
Next: [[Homelab/Software/OpenWrt/Installation/Pt 2, Flashing-and-booting|Pt 2, Flashing and booting]]
