---
title: Lenovo ThinkPad P52s Gotchas
description: Hardware-rooted problems hit while turning the ThinkPad P52s into the home lab router.
tags:
  - Homelab
  - Hardware
  - Gotchas
---
# NVMe drive won't boot after the transplant

After I moved the [[Homelab/Hardware/Acer-Aspire-3/Acer-aspire-3-overview|Acer's]] NVMe drive into this ThinkPad, the UEFI boot menu listed the drive, but selecting it just flashed black and returned to the menu.

![[nvme-drive-fails-to-boot-on-thinkpad.gif]]
*The NVMe drive failing to boot on the ThinkPad — selecting it drops straight back to the boot menu.*

**Cause:** [[Homelab/Hardware/Lenovo-Thinkpad-P52s/Lenovo-thinkpad-p52s-glossary#Secure Boot|Secure Boot]] was blocking the unsigned OpenWrt bootloader.

**Fix:** Toggling the "Secure Boot" setting to disabled in the UEFI menu was *not* sufficient — the setting didn't actually take effect on save/exit. It only took effect once I wiped the Secure Boot keys from the UEFI menu entirely. I still don't know why toggling alone doesn't disable enforcement on this firmware.

---
# Only one Wi-Fi radio: can't be a Wi-Fi client and host an AP at the same time

Once I had OpenWrt connected to the building's WPA2-EAP Wi-Fi (see [[Homelab/Software/OpenWrt/Enterprise-wifi-authentication|Enterprise Wi-Fi authentication]]), I created a second wireless network (WPA2-PSK, its own passphrase) so a phone could join the LAN directly, attached to the LAN interface. Turning that access point on caused the phone to connect but report no internet, and shortly after, OpenWrt itself dropped off the building's Wi-Fi entirely.

![[phone-connected-to-custom-ap-with-no-internet.png]]
*The phone connects to the new AP but has no internet — the first sign something was wrong.*

![[openwrt-disconnected-from-building-wifi-after-enabling-ap.png]]
*OpenWrt loses its connection to the building's Wi-Fi as soon as the AP is turned on.*

I spent about an hour searching online without finding anything useful, then spent several more hours working through the problem with Claude, which correctly diagnosed the underlying cause.

**Cause:** the ThinkPad has a single physical Wi-Fi radio. That radio can only sit on one channel/band at a time. One symptom of this: the hosted network would only come up on 2.4GHz, since 5GHz has radar-detection compliance rules that apply to broadcast networks but not to client connections. Beyond that, hosting the AP forces the radio to reconfigure — including switching bands — which knocks the already-established client connection off its channel. The client side would then fail to reconnect at all, and only a full reboot restored it.

This is a hardware constraint of the laptop's Wi-Fi card, not a software bug: the same conflict would occur under any router OS installed on this same physical radio (the [[Homelab/Hardware/Lenovo-Thinkpad-P52s/Lenovo-thinkpad-p52s-glossary#Wi-Fi radio|OS-agnostic test]] — a different OS on the same single-radio hardware would hit the same limitation).

Workarounds I tried that did **not** hold up:
- Locking both networks to the same fixed channel — the setting only applies to the broadcast network, not the outbound client connection, which keeps roaming freely.
- Pinning the outbound connection to one specific access point instead of letting it roam — reduced but didn't eliminate the interference.

**Resolution:** none yet — I need a second, dedicated device to host the local AP. The custom AP feature isn't usable on this hardware as configured. This is a genuine hardware limitation, not something to keep trying to fix in OpenWrt's Wi-Fi config.
