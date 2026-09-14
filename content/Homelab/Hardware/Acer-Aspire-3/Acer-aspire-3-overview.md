---
title: Acer Aspire 3 Overview
description: The laptop originally chosen to become the home lab's router, and why it was retired from that role before ever going into service.
tags:
  - Homelab
  - Hardware
---
# What it is

An Acer Aspire 3 A315-24PT laptop with an AMD Ryzen 5 7520U (2.8GHz), 8GB of RAM, and a 512GB NVMe SSD. I picked it as the router candidate because the dorm/apartment I built the home lab in has no Ethernet port at all — every device has to reach the internet over the building's enterprise (WPA2-EAP) Wi-Fi, and running a dedicated router lets me keep that Wi-Fi login in one place instead of configuring it on every server.

I installed [[Homelab/Software/OpenWrt/Installation/Pt 1, Choosing-the-router-os|OpenWrt]] onto this laptop's internal NVMe drive as the router OS (see the [[Homelab/Software/OpenWrt/Installation/Pt 2, Flashing-and-booting|installation walkthrough]] for how).

---
# Why I retired it from router duty

After I successfully installed OpenWrt to the internal drive, it became clear the laptop **has no Ethernet port** — only Wi-Fi. A router needs a wired LAN side to hand out to the rest of the home lab, so the Acer couldn't do the job as-is.

# The NVMe transplant

Rather than reinstalling OpenWrt from scratch on a different machine, I pulled the Acer's NVMe drive (already containing a working OpenWrt install) and moved it into the [[Homelab/Hardware/Lenovo-Thinkpad-P52s/Lenovo-thinkpad-p52s-overview|Lenovo ThinkPad P52s]], which does have an Ethernet port. The drive wasn't soldered to the board, which made this possible — I confirmed that by opening the case with an [iFixit "Jimmy" opening tool](https://www.ifixit.com/products/jimmy).

I left the internal battery disconnected during the teardown; it plays no further role since the Acer is no longer in service as a router.
