---
title: Lenovo ThinkPad P52s Overview
description: The laptop that actually became the home lab router, after the Acer Aspire 3 turned out to have no Ethernet port.
tags:
  - Homelab
  - Hardware
---
# What it is

A donated Lenovo ThinkPad P52s (model 20LB0026US) with an i7-8550U CPU and 16GB of RAM. It arrived with its storage drive removed by the previous owner, so it had no OS and no storage of its own.

# Becoming the router

Since the [[Homelab/Hardware/Acer-Aspire-3/Acer-aspire-3-overview|Acer Aspire 3]] already had a working [[Homelab/Software/OpenWrt/Installation/Pt 1, Choosing-the-router-os|OpenWrt]] install but turned out to have no Ethernet port, I moved its NVMe drive into this ThinkPad instead.

The ThinkPad's NVMe slot isn't obvious from a top-down view of the motherboard — it sits underneath a large metal shield in the bottom-left of the case (likely reused from an older model that put a 2.5" SSD in that spot, with a newer NVMe adapter substituted in during a later revision).

![[thinkpad-p52s-nvme-slot-hidden-under-metal-shield.png]]
*The NVMe slot's actual location, hidden under a metal shield — not visible without knowing to look under it.*

I removed the internal (second) battery while the case was open, so the router now runs wall-power only.

---
# Gotchas

See [[Homelab/Hardware/Lenovo-Thinkpad-P52s/Lenovo-thinkpad-p52s-gotchas|Gotchas]] for the Secure Boot boot failure and the single Wi-Fi radio limitation I discovered while bringing this laptop into service as the router.
