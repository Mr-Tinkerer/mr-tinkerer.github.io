---
title: Dell Vostro 260s Gotchas
description: Physical/firmware problems discovered after the NAS was moved into its permanent spot.
tags:
  - Homelab
  - Hardware
  - Gotchas
---
# NAS unreachable after being moved into place

After I installed [[Homelab/Software/OpenMediaVault/Openmediavault-installation|OpenMediaVault]] and moved the machine (monitor disconnected) to its permanent spot, its web UI became unreachable. Pulling the machine back out and reattaching a monitor showed it was stuck at the POST screen, waiting for a keypress to continue booting.

![[nas-post-screen-waiting-for-keypress.png]]
*The NAS stuck at POST, waiting for a keyboard press it will never get once deployed headless.*

**Fix:** I disabled the "wait for keypress on error/no keyboard" option in the BIOS, so the machine boots straight through with no keyboard attached.

![[nas-boots-headless-after-disabling-post-wait.png]]
*The NAS booting straight into OpenMediaVault with no keyboard attached, after the BIOS setting was disabled.*

# BIOS settings resetting after a long power-off

My fix above didn't stick permanently — after the machine was powered off for an extended period, the BIOS reverted to waiting for a keypress again. This is the classic symptom of a dead or missing [[Homelab/Hardware/Dell-Vostro-260s/Dell-vostro-260s-glossary#CMOS battery|CMOS battery]] failing to hold settings without wall power. Swapping the CMOS battery on the motherboard resolved it.
