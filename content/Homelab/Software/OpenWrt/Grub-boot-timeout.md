---
title: OpenWrt GRUB Boot Timeout
description: Fixing GRUB waiting forever for Enter instead of auto-booting after a few seconds.
tags:
  - Homelab
  - OpenWrt
---

## Overview

This page covers fixing GRUB waiting forever for Enter instead of auto-booting after a few seconds on OpenWrt.

## Problem

GRUB was not timing out to the default boot entry — it required manual Enter every time the router booted, defeating the point of an unattended router.

### Configuration File Location

Most GRUB timeout guides assume `/etc/default/grub` (Debian-style) or `/etc/config/grub` (OpenWrt's UCI-based config, only present on squashfs images). Neither file existed on the ext4-installed OpenWrt image, so the generated boot config (`/etc/boot/grub.cfg`) was edited directly instead.

## Root Cause

OpenWrt's default `grub.cfg` inserts these two lines:

```bash
serial --unit=0 --speed=115200 --word=8 --parity=no --stop=1 --rtscts=off
terminal_input console serial; terminal_output console serial
```

The second line tells GRUB to treat both the console and the serial port as valid input/output. Since this laptop has no serial connection, GRUB waits indefinitely for serial input that will never arrive, and the timer never starts.

## Fix

### terminal Configuration

Replace those two lines with:

```bash
terminal_input console
terminal_output console
```

### Timeout Setting

Set `timeout="0"` (down from the default) to skip straight to the kernel instead of waiting on the boot menu at all.
