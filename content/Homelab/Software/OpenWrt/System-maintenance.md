---
title: System maintenance
description: Password/timezone housekeeping, package updates, an SSH-over-Kitty fix, and expanding the root filesystem to use the full disk.
tags:
  - Homelab
  - OpenWrt
---
# Routine housekeeping

- I changed the router (LuCI) admin password from unset to a real password under `System → Administration → Router Password`.
- I set the timezone correctly under `System → System → General Settings`.
- I refreshed the package lists and applied all available package upgrades under `System → Software`.

# Kitty terminal SSH failure — not actually fixed

SSH connections from the Kitty terminal emulator initially failed for me. Installing the `coreutils-base64` package on OpenWrt (Kitty's SSH integration depends on a `base64` binary being present on the remote end, which OpenWrt's minimal image doesn't ship by default) fixed that specific failure — but it wasn't the end of it. A later OpenWrt update broke keyboard input display over SSH from Kitty in a different way: I can still type and the remote end receives my input, but I can't see what I'm typing on screen.

Between that and other recurring annoyances (Kitty not properly opening files), I switched to XFCE Terminal for SSH into this router instead. No issues since.

# Root filesystem only using a fraction of the disk

I noticed OpenWrt was only reporting ~100MB used out of the laptop's 512GB drive, since the `ext4` install only creates a partition sized to the image, not the whole disk. I fixed it by booting a live Linux environment and resizing the `ext4` partition to consume the rest of the disk.

![[openwrt-ext4-partition-resized-to-full-disk.png]]
*The `ext4` partition after being resized to use the laptop's full storage capacity.*
