---
title: Build and repairs
description: The case, power supply, motherboard, and drive-sled problems hit while rebuilding the Vostro 260s into a NAS.
tags:
  - Homelab
  - Hardware
---
# The Ciara case didn't work out

The donated "Ciara" case (exact model unknown) had a pre-installed generic 400W Power Man PSU and room for hard drive bays, but I hit two problems immediately:

- **No hard drive sleds.** The bays had nothing to actually hold a drive in place.

  ![[ciara-case-missing-hard-drive-sleds.png]]
  *The Ciara case's hard-drive bay has no sleds — drives can't be secured in it as-is.*

- **PSU CPU cable too short.** The CPU power connector on the pre-installed PSU could not physically reach the CPU socket at the top of the case.

  ![[psu-cpu-cable-too-short-for-case.png]]
  *The PSU's CPU power cable stretched as far as it goes — about halfway up the board, well short of the socket.*

I later replaced the PSU with a semi-modular 850W 80+ Bronze Antec unit whose cable reached fine, and swapped the motherboard itself for one with more SATA ports and RAM slots. One screw held by a stuck motherboard standoff had to be worked loose during that swap:

![[vostro-motherboard-standoff-stuck-in-screw.png]]
*A screw that wouldn't turn — the motherboard standoff underneath was unscrewing along with it, not the screw itself.*

# Hard drive sleds: three attempts

1. Individual clip-on sleds I borrowed from the school kept popping back off the case when sliding the drive in.

   ![[individual-hdd-sled-fails-to-stay-attached.png]]
   *A borrowed sled attached to a drive — it would not stay seated in the case's bay.*

2. Sleds I bought online turned out to be 2.5"-to-3.5" adapters, not case-mount sleds, and also didn't fit.
3. I eventually abandoned the Ciara case in favor of a **different, $5 case** I bought off Facebook Marketplace that already included matching sleds, 4 drive bays, and a spare optical bay.

With the new case's own sleds, and no way to open its rear panel to route cables normally, I connected the drives via an improvised routing scheme: I fed cables through an oval cutout, connected each drive's data + power cable while the drive was only halfway seated (to keep reach to the connectors), and slid all the drives fully home together once wired.

![[nas-case-final-drive-cable-routing.png]]
*The final cable routing inside the $5 case — data and power run through a cutout to reach drives that had to be wired while half-inserted.*

I confirmed all four bays plus the recovered Kingston 120GB boot SSD were detected in the BIOS afterward:

![[nas-case-all-drives-detected-in-bios.png]]
*All drives, including the recovered boot SSD, detected correctly in the BIOS after the rebuild.*

With the case, PSU, motherboard, and drives all in place, I [[Homelab/Software/OpenMediaVault/Openmediavault-installation|installed OpenMediaVault]] as the NAS OS.
