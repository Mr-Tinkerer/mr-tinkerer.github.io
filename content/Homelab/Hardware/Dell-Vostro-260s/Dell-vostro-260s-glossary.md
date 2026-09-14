---
title: Dell Vostro 260s Glossary
description: Terms used across the Dell Vostro 260s / NAS bucket.
tags:
  - glossary
---
## CMOS battery

A small coin-cell battery on the motherboard that keeps BIOS/UEFI settings and the system clock alive while the PC is unplugged from wall power. When it dies or is missing, the board loses those saved settings any time it's fully de-powered for a while, and BIOS options silently revert to their defaults — which is exactly what I ran into on this NAS: after it sat powered off for an extended stretch, my earlier "skip wait for keypress" fix reverted on its own, and swapping the CMOS battery was what finally made it stick. See [CMOS battery (Wikipedia)](https://en.wikipedia.org/wiki/Nonvolatile_BIOS_memory).
