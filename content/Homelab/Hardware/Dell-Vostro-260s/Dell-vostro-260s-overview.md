---
title: Dell Vostro 260s Overview
description: A retired Dell PC whose ATX motherboard was moved across two different cases to become the home lab's NAS.
tags:
  - Homelab
  - Hardware
---
# What it is

A second free retired PC from the same college IT department as the [[Homelab/Hardware/Dell-Optiplex-5060/Dell-optiplex-5060-overview|Dell Optiplex 5060]]. Unlike the Optiplex, the Vostro 260s uses a standard ATX motherboard, so I could pull it out of its original case entirely and rebuild it into a case with more 3.5" drive bays — the point being to use this machine as the home lab's [[Homelab/Software/OpenMediaVault/Openmediavault-installation|OpenMediaVault]] NAS.

It came with two 4GB DDR3 sticks (different brands, mixed without issue) and the same model Seagate 500GB drive as the Optiplex.

# A "ship of Theseus" rebuild

By the time I finished the NAS, I had replaced effectively every major part of the original Vostro:

- **Case:** the stock Vostro case → a donated "Ciara" case (no room for the CPU power cable to reach, no hard-drive sleds) → a $5 Facebook Marketplace case with 4 drive bays and an empty optical bay.
- **Power supply:** the stock unit → a non-modular 400W 80+ Power Man unit (too short a CPU cable) → a semi-modular 850W 80+ Bronze Antec unit.
- **Motherboard:** the original Vostro board (2 DDR3 slots, 4 SATA ports) → a different board with 4 DDR3 slots (2 pre-populated with free RAM) and 6 SATA ports.
- **Boot drive:** a Kingston SSDNow V+200 120GB SATA SSD, which I found already installed in the $5 case and kept as the boot drive instead of buying one.

Only the name "Dell Vostro 260s" still identifies this machine — none of its core hardware from the original PC is still installed. See [[Homelab/Hardware/Dell-Vostro-260s/Build-and-repairs|Build and repairs]] for the full sequence of swaps and the problems I hit along the way, and [[Homelab/Hardware/Dell-Vostro-260s/Dell-vostro-260s-gotchas|Gotchas]] for the post-install physical issues.
