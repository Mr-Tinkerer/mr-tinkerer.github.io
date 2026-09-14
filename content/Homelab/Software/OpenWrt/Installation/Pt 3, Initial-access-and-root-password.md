---
title: "Pt 3: Initial access and root password"
description: Discovering the router laptop had no Ethernet port, transplanting the install, and the quirks of first login.
tags:
  - Homelab
  - OpenWrt
---
# Prerequisites

- [[Homelab/Software/OpenWrt/Installation/Pt 1, Choosing-the-router-os|Pt 1, Choosing the router OS]]
- [[Homelab/Software/OpenWrt/Installation/Pt 2, Flashing-and-booting|Pt 2, Flashing and booting]] — OpenWrt installed to the laptop's internal drive and booting successfully.

---
# No Ethernet port

The [[Homelab/Hardware/Acer-Aspire-3/Acer-aspire-3-overview|Acer Aspire 3]] laptop I'd just installed OpenWrt onto turned out to have no Ethernet port at all — only Wi-Fi. Since a router needs a wired LAN side, I moved its NVMe drive (already containing the working OpenWrt install) into the [[Homelab/Hardware/Lenovo-Thinkpad-P52s/Lenovo-thinkpad-p52s-overview|Lenovo ThinkPad P52s]] instead. See the ThinkPad's [[Homelab/Hardware/Lenovo-Thinkpad-P52s/Lenovo-thinkpad-p52s-gotchas|Secure Boot gotcha]] for the boot failure I hit during that move.

# First login to LuCI

With both the router and a management machine connected to the same switch, `https://192.168.1.1` (OpenWrt's default LAN address) opened the [[Homelab/Software/OpenWrt/Openwrt-glossary#LuCI|LuCI]] web UI login page.

Since the root account had no password set yet (this was a fresh install after the transplant/reinstall), **any password I entered at the login screen was accepted** — LuCI lets you into the root account with any input until a root password is explicitly set. After logging in this way, I set a real password immediately from `System → Administration`.

---
This completes the base install. See [[Homelab/Software/OpenWrt/Wifi-driver-installation|Wi-Fi driver installation]] next.
