---
title: Dell Vostro 260s Build and Repairs
description: The case, power supply, motherboard, and drive-sled problems hit while rebuilding the Vostro 260s into a NAS.
tags:
  - Homelab
  - Hardware
---

## Overview

This page documents the case, power supply, motherboard, and drive-sled problems hit while rebuilding the Vostro 260s into a NAS.

## The Ciara Case Didn't Work Out

The donated "Ciara" case (exact model unknown) had a pre-installed generic 400W Power Man PSU and room for hard drive bays, but two problems were hit immediately:

### No Hard Drive Sleds

The bays had nothing to actually hold a drive in place.

See [missing sleds screenshot](Images/ciara-case-missing-hard-drive-sleds.png) for the Ciara case's hard-drive bay having no sleds — drives can't be secured in it as-is.

### PSU CPU Cable Too Short

The CPU power connector on the pre-installed PSU could not physically reach the CPU socket at the top of the case.

See [short cable screenshot](Images/psu-cpu-cable-too-short-for-case.png) for the PSU's CPU power cable stretched as far as it goes — about halfway up the board, well short of the socket.

The PSU was later replaced with a semi-modular 850W 80+ Bronze Antec unit whose cable reached fine, and the motherboard itself was swapped for one with more SATA ports and RAM slots. One screw held by a stuck motherboard standoff had to be worked loose during that swap:

See [stuck standoff screenshot](Images/vostro-motherboard-standoff-stuck-in-screw.png) for a screw that wouldn't turn — the motherboard standoff underneath was unscrewing along with it, not the screw itself.

## Hard Drive Sleds: Three Attempts

1. **Individual clip-on sleds borrowed from the school** kept popping back off the case when sliding the drive in.

   See [borrowed sled fails screenshot](Images/individual-hdd-sled-fails-to-stay-attached.png) for a borrowed sled attached to a drive — it wouldn't stay seated in the case's bay.

2. **Sleds bought online** turned out to be 2.5"-to-3.5" adapters, not case-mount sleds, and also didn't fit.

3. **The Ciara case was abandoned** in favor of a different, $5 case bought off Facebook Marketplace that already included matching sleds, 4 drive bays, and a spare optical bay.

With the new case's own sleds, and no way to open its rear panel to route cables normally, the drives were connected via an improvised routing scheme: cables were fed through an oval cutout, each drive's data + power cable was connected while the drive was only halfway seated (to keep reach to the connectors), and all the drives were slid fully home together once wired.

See [final routing screenshot](Images/nas-case-final-drive-cable-routing.png) for the final cable routing inside the $5 case — data and power run through a cutout to reach drives that had to be wired while half-inserted.

All four bays plus the recovered Kingston 120GB boot SSD were confirmed to be detected in the BIOS afterward:

See [BIOS detection screenshot](Images/nas-case-all-drives-detected-in-bios.png) for all drives, including the recovered boot SSD, detected correctly in the BIOS after the rebuild.

With the case, PSU, motherboard, and drives all in place, OpenMediaVault was installed as the NAS OS.
