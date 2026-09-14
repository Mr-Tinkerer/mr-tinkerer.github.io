---
title: "Pt 2: Flashing and booting"
description: Getting the OpenWrt image onto a USB stick, discovering the .img file is the OS itself (not an installer), and writing it to internal storage.
tags:
  - Homelab
  - OpenWrt
---
# Prerequisites

- [[Homelab/Software/OpenWrt/Installation/Pt 1, Choosing-the-router-os|Pt 1, Choosing the router OS]] — OpenWrt selected, `.img.gz` downloaded.

---
# Ventoy attempt (failed)

I first copied the extracted `.img` to an existing [Ventoy](https://www.ventoy.net/) USB stick. Booting it produced an error that Ventoy's own docs explain: OpenWrt images don't ship the kernel modules Ventoy needs, so a separate `ventoy_openwrt.xz` plugin file has to be placed — **unextracted** — in a `ventoy` folder on the stick.

![[ventoy-openwrt-boot-error-missing-kernel-modules.png]]
*The Ventoy boot error caused by the missing kernel-module plugin.*

Even after I placed the plugin file (first extracted by mistake, then correctly left as the `.xz`), OpenWrt would boot partway, print kernel messages, then reboot itself — the log showed `/dev/ventoy2: Can't lookup blockdev`. I abandoned Ventoy as the boot medium for this image.

# Direct flash with Fedora Media Writer

I instead flashed the `.img` directly to a second USB stick using Fedora Media Writer. The stick then didn't appear at all in the boot menu — because the laptop's "BIOS" screen was actually a UEFI menu, and the image I'd initially downloaded was the **non-EFI** variant. Re-flashing with the **EFI** image fixed this.

# Discovering `.img` is the OS, not an installer

Booting the EFI image got me as far as a shell prompt (after I learned that OpenWrt's install boot sequence pauses on `Please press Enter to activate this console`). At this point it became clear, via [OpenWrt's x86 install guide](https://openwrt.org/docs/guide-user/installation/openwrt_x86), that **the `.img` file already *is* the OS** — flashing it to a USB stick installs OpenWrt *to that USB stick*, it does not produce an installer for internal storage.

To get OpenWrt onto the laptop's internal drive without removing it, I used the guide's live-environment method: boot a general-purpose Linux live environment (Arch Linux, already on hand), mount the USB stick holding the `.img`, and `dd` the image onto the internal disk.

After that, the laptop booted OpenWrt directly from internal storage with no USB media attached — the OS install itself was complete at this point.

---
Next: [[Homelab/Software/OpenWrt/Installation/Pt 3, Initial-access-and-root-password|Pt 3, Initial access and root password]]
