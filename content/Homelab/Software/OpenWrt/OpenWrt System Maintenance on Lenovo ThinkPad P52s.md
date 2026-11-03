---
title: OpenWrt System Maintenance on Lenovo ThinkPad P52s
description: Password/timezone housekeeping, package updates, an SSH-over-Kitty fix, and expanding the root filesystem to use the full disk.
tags:
  - Homelab
  - OpenWrt
  - Maintenance
---

## Overview

This page covers routine OpenWrt maintenance tasks including password changes, timezone configuration, package updates, SSH terminal issues, and filesystem expansion.

## Routine Housekeeping

- **Admin Password** — Changed the router (LuCI) admin password from unset to a real password under `System → Administration → Router Password`
- **Timezone** — Set the timezone correctly under `System → System → General Settings`
- **Package Updates** — Refreshed package lists and applied all available package upgrades under `System → Software`

## Kitty Terminal SSH Failure

SSH connections from the Kitty terminal emulator initially failed. Installing the `coreutils-base64` package on OpenWrt fixed this — Kitty's SSH integration depends on a `base64` binary being present on the remote end, which OpenWrt's minimal image doesn't ship by default.

### Ongoing Issue

A later OpenWrt update broke keyboard input display over SSH from Kitty in a different way: input is received correctly, but typed characters don't display on screen. Between this and other recurring annoyances (Kitty not properly opening files), XFCE Terminal was switched to for SSH into this router. No issues since.

## Root Filesystem Underutilization

OpenWrt was only reporting ~100MB used out of the laptop's 512GB drive, since the `ext4` install creates a partition sized to the image, not the whole disk.

### Resolution

A live Linux environment was booted and used to resize the `ext4` partition to consume the rest of the disk.

See [ext4 partition resized screenshot](Images/openwrt-ext4-partition-resized-to-full-disk.png) for the `ext4` partition after being resized to use the laptop's full storage capacity.
