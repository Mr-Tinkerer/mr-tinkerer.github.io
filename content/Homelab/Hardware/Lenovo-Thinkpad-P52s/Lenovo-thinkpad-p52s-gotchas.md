---
title: Lenovo ThinkPad P52s Gotchas
description: Hardware-rooted problems hit while turning the ThinkPad P52s into the home lab router.
tags:
  - Homelab
  - Hardware
  - Gotchas
---

## Overview

This page documents hardware-rooted problems hit while turning the ThinkPad P52s into the home lab router.

## NVMe Drive Won't Boot After the Transplant

After the Acer's NVMe drive was moved into this ThinkPad, the UEFI boot menu listed the drive, but selecting it just flashed black and returned to the menu.

See [NVMe drive fails GIF](Images/nvme-drive-fails-to-boot-on-thinkpad.gif) for the NVMe drive failing to boot on the ThinkPad — selecting it drops straight back to the boot menu.

### Cause

Secure Boot was blocking the unsigned OpenWrt bootloader.

### Fix

Toggling the "Secure Boot" setting to disabled in the UEFI menu was not sufficient — the setting didn't actually take effect on save/exit. It only took effect once the Secure Boot keys were wiped from the UEFI menu entirely. Why toggling alone doesn't disable enforcement on this firmware is still unknown.

## Only One Wi-Fi Radio: Can't Be a Wi-Fi Client and Host an AP at the Same Time

Once OpenWrt was connected to the building's WPA2-EAP Wi-Fi, a second wireless network (WPA2-PSK, its own passphrase) was created so a phone could join the LAN directly, attached to the LAN interface. Turning that access point on caused the phone to connect but report no internet, and shortly after, OpenWrt itself dropped off the building's Wi-Fi entirely.

See [phone connected screenshot](Images/phone-connected-to-custom-ap-with-no-internet.png) for the phone connecting to the new AP but having no internet — the first sign something was wrong.

See [OpenWrt disconnected screenshot](Images/openwrt-disconnected-from-building-wifi-after-enabling-ap.png) for OpenWrt losing its connection to the building's Wi-Fi as soon as the AP was turned on.

An hour searching online without finding anything useful was spent, then several more hours working through the problem with Claude, which correctly diagnosed the underlying cause.

### Cause

The ThinkPad has a single physical Wi-Fi radio. That radio can only sit on one channel/band at a time. One symptom: the hosted network would only come up on 2.4GHz, since 5GHz has radar-detection compliance rules that apply to broadcast networks but not to client connections. Beyond that, hosting the AP forces the radio to reconfigure — including switching bands — which knocks the already-established client connection off its channel. The client side would then fail to reconnect at all, and only a full reboot restored it.

This is a hardware constraint of the laptop's Wi-Fi card, not a software bug: the same conflict would occur under any router OS installed on this same physical radio (the OS-agnostic test — a different OS on the same single-radio hardware would hit the same limitation).

### Workarounds Tried (That Did Not Hold Up)

- **Locking both networks to the same fixed channel** — The setting only applies to the broadcast network, not the outbound client connection, which keeps roaming freely
- **Pinning the outbound connection to one specific access point instead of letting it roam** — Reduced but didn't eliminate the interference

### Resolution

None yet — a second, dedicated device is needed to host the local AP. The custom AP feature isn't usable on this hardware as configured. This is a genuine hardware limitation, not something to keep trying to fix in OpenWrt's Wi-Fi config.
