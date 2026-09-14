---
title: GRUB boot timeout
description: Fixing GRUB waiting forever for Enter instead of auto-booting after a few seconds.
tags:
  - Homelab
  - OpenWrt
---
# The problem

I found GRUB was not timing out to the default boot entry — it required me to manually press Enter every time the router booted, defeating the point of an unattended router.

Most GRUB timeout guides assume `/etc/default/grub` (Debian-style) or `/etc/config/grub` (OpenWrt's UCI-based config, only present on [[Homelab/Software/OpenWrt/Openwrt-glossary#SquashFS|squashfs]] images). Neither file existed on my `ext4`-installed OpenWrt image, so I had to edit the generated boot config directly instead: `/etc/boot/grub.cfg`.

I turned to Google's Gemini to help track down the actual fix once those two standard config paths turned out to be dead ends.

# Root cause

OpenWrt's default `grub.cfg` inserts these two lines:

```bash
serial --unit=0 --speed=115200 --word=8 --parity=no --stop=1 --rtscts=off
terminal_input console serial; terminal_output console serial
```

The second line tells GRUB to treat *both* the console and the serial port as valid input/output. Since this laptop has no serial connection, GRUB waits indefinitely for serial input that will never arrive, and the timer never starts.

# Fix

I replaced those two lines with:

```bash
terminal_input console
terminal_output console
```

and set `timeout="0"` (down from the default) to skip straight to the kernel instead of waiting on the boot menu at all.
