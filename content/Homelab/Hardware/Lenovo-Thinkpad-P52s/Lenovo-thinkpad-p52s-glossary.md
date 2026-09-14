---
title: Lenovo ThinkPad P52s Glossary
description: Terms used across the Lenovo ThinkPad P52s bucket.
tags:
  - glossary
---
## Secure Boot

A UEFI feature that only allows cryptographically signed bootloaders to run, refusing to boot anything else. I ran into it directly on this ThinkPad: disabling the Secure Boot *setting* in firmware wasn't enough on its own to actually turn off enforcement — I had to wipe the Secure Boot keys from the UEFI menu entirely before the transplanted OpenWrt drive would boot. See [[Homelab/Hardware/Lenovo-Thinkpad-P52s/Lenovo-thinkpad-p52s-gotchas|the ThinkPad's Secure Boot gotcha]]. Reference: [Secure Boot (Wikipedia)](https://en.wikipedia.org/wiki/UEFI#Secure_Boot).

## Wi-Fi radio

The physical transceiver hardware inside a device that sends and receives Wi-Fi signals. A device with a single radio can only be tuned to one channel/band at a time, which limits it to one Wi-Fi "role" (client or access point) at once unless it supports true concurrent dual-role operation. This is the root cause behind the ThinkPad's AP conflict: it only has one Wi-Fi radio, so it can't stay connected to the building's Wi-Fi as a client while also broadcasting its own access point — turning on the AP forces the single radio to reconfigure and drops the existing client connection. See [[Homelab/Hardware/Lenovo-Thinkpad-P52s/Lenovo-thinkpad-p52s-gotchas|the single-radio AP+client conflict]].
