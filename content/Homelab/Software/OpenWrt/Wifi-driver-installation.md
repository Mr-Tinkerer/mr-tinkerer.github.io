---
title: Wi-Fi driver installation
description: Getting OpenWrt to recognize the ThinkPad's Intel 8265 Wi-Fi card, offline.
tags:
  - Homelab
  - OpenWrt
---
# The problem

After [[Homelab/Software/OpenWrt/Installation/Pt 3, Initial-access-and-root-password|initial setup]], `Network → Wireless` didn't appear in LuCI at all — OpenWrt only shows that tab once it detects a wireless device, and the built-in Wi-Fi card wasn't being detected. I confirmed the laptop's Wi-Fi card does work under Linux by booting a live Ubuntu 22.04 environment and successfully using it on kernel `6.8.0`, older than OpenWrt's own `6.12.94`:

![[ubuntu-2204-live-environment-wifi-working-older-kernel.png]]
*Ubuntu 22.04's live environment using the Wi-Fi chip fine on an older kernel than OpenWrt ships — confirming the chip works under Linux, so the driver just isn't bundled in OpenWrt's default image.*

![[openwrt-wifi-card-missing-from-interface-dropdown.png]]
*The Wi-Fi device missing from OpenWrt's own interface device list, before the driver packages were installed.*

This is specific to OpenWrt's minimal default image, not a hardware fault — the [[Homelab/Hardware/Lenovo-Thinkpad-P52s/Lenovo-thinkpad-p52s-overview|same laptop]] runs the chip fine under a general-purpose distro.

# Identifying the card

Since OpenWrt lacks `lshw`, I used a live Fedora boot instead to run `lshw -c network`, which identified the chip as an Intel Dual Band Wireless-AC 8265/8275:

![[lshw-output-identifying-intel-8265-wifi-chip.png]]
*`lshw -c network` output identifying the exact Wi-Fi chip model — the only reliable way to know which firmware/driver packages to grab.*

# Installing packages offline

Since the router had no working internet connection yet, I had to download packages on another machine and copy them over by USB (or `scp`). I read the repository index from `/etc/apk/repositories.d/distfeeds.list` to find package URLs. I installed packages with `apk add *.apk`, adding `--allow-untrusted` since the manually-downloaded `.apk` files aren't signed by a trusted key in this offline flow.

I had to resolve dependencies by hand, one missing-dependency error at a time, until I had this full package set installed:

- `hostapd-common`, `intel-microcode`, `iw-full`, `iwlwifi-firmware-iwl8265`, `kmod-cfg80211`, `kmod-crypto-aead`, `kmod-crypto-ccm`, `kmod-crypto-cmac`, `kmod-crypto-ctr`, `kmod-crypto-gcm`, `kmod-crypto-geniv`, `kmod-crypto-gf128`, `kmod-crypto-ghash`, `kmod-crypto-hmac`, `kmod-crypto-manager`, `kmod-crypto-null`, `kmod-crypto-rng`, `kmod-crypto-seqiv`, `kmod-crypto-sha3`, `kmod-crypto-sha512`, `kmod-iwlwifi`, `kmod-mac80211`, `ucode-mod-digest`, `ucode-mod-nl80211`, `ucode-mod-rtnl`, `wifi-scripts`, `wireless-regdb`

After installing all of these and rebooting, the Wireless tab appeared:

![[openwrt-wireless-tab-appears-after-driver-install.png]]
*The Wireless tab now available in LuCI after installing the Intel 8265 firmware and supporting kernel modules.*

Connecting to a network also required the `wpa_supplicant` package, which is separate from the driver packages above — see [[Homelab/Software/OpenWrt/Enterprise-wifi-authentication|Enterprise Wi-Fi authentication]].
