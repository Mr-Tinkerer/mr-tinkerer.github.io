---
title: OpenWrt Wi-Fi Driver Installation on Lenovo ThinkPad P52s
description: Getting OpenWrt to recognize the ThinkPad's Intel 8265 Wi-Fi card, offline.
tags:
  - Homelab
  - OpenWrt
  - Networking
---

## Overview

This guide covers installing the Intel 8265 Wi-Fi driver on OpenWrt when the router has no working internet connection, requiring offline package installation.

## Problem Description

After initial setup, the `Network → Wireless` tab didn't appear in LuCI at all — OpenWrt only shows that tab once it detects a wireless device, and the built-in Wi-Fi card wasn't being detected.

### Hardware Verification

The laptop's Wi-Fi card works under Linux by booting a live Ubuntu 22.04 environment and successfully using it on kernel `6.8.0`, older than OpenWrt's own `6.12.94`.

See [Ubuntu live environment screenshot](Images/ubuntu-2204-live-environment-wifi-working-older-kernel.png) for the Ubuntu 22.04 live environment using the Wi-Fi chip fine on an older kernel than OpenWrt ships — confirming the chip works under Linux.

See [missing device screenshot](Images/openwrt-wifi-card-missing-from-interface-dropdown.png) for the Wi-Fi device missing from OpenWrt's own interface device list before driver packages were installed.

### Root Cause

This is specific to OpenWrt's minimal default image, not a hardware fault — the same laptop runs the chip fine under a general-purpose distro.

## Identifying the Card

Since OpenWrt lacks `lshw`, a live Fedora boot was used to run `lshw -c network`, which identified the chip as an Intel Dual Band Wireless-AC 8265/8275.

See [lshw output screenshot](Images/lshw-output-identifying-intel-8265-wifi-chip.png) for the `lshw -c network` output identifying the exact Wi-Fi chip model — the only reliable way to know which firmware/driver packages to grab.

## Offline Package Installation

### Package Transfer

Since the router had no working internet connection yet, packages were downloaded on another machine and copied over by USB or `scp`.

### Package Discovery

The repository index was read from `/etc/apk/repositories.d/distfeeds.list` to find package URLs. Packages were installed with `apk add *.apk`, adding `--allow-untrusted` since the manually-downloaded `.apk` files aren't signed by a trusted key in this offline flow.

### Package Set

Dependencies were resolved by hand, one missing-dependency error at a time, until the following full package set was installed:

- `hostapd-common`
- `intel-microcode`
- `iw-full`
- `iwlwifi-firmware-iwl8265`
- `kmod-cfg80211`
- `kmod-crypto-aead`
- `kmod-crypto-ccm`
- `kmod-crypto-cmac`
- `kmod-crypto-ctr`
- `kmod-crypto-geniv`
- `kmod-crypto-gf128`
- `kmod-crypto-ghash`
- `kmod-crypto-hmac`
- `kmod-crypto-manager`
- `kmod-crypto-null`
- `kmod-crypto-rng`
- `kmod-crypto-seqiv`
- `kmod-crypto-sha3`
- `kmod-crypto-sha512`
- `kmod-iwlwifi`
- `kmod-mac80211`
- `ucode-mod-digest`
- `ucode-mod-nl80211`
- `ucode-mod-rtnl`
- `wifi-scripts`
- `wireless-regdb`

### Reboot and Verification

After installing all packages and rebooting, the Wireless tab appeared in LuCI.

See [wireless tab screenshot](Images/openwrt-wireless-tab-appears-after-driver-install.png) for the Wireless tab now available after installing the Intel 8265 firmware and supporting kernel modules.

### wpa_supplicant Requirement

Connecting to a network also requires the `wpa_supplicant` package, which is separate from the driver packages above — see the Enterprise Wi-Fi authentication page.
