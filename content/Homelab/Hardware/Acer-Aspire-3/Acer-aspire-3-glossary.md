---
title: Acer Aspire 3 Glossary
description: Terms used across the Acer Aspire 3 bucket.
tags:
  - glossary
---
## UEFI

The firmware interface that replaced legacy BIOS on modern PCs. Unlike BIOS, it can boot directly from GPT disks, supports larger boot drives, and separates "UEFI mode" boot entries from legacy ones in its boot menu. It matters here because it's the layer I have to work through any time I'm choosing which drive a machine should boot from — including when I moved the Acer's NVMe drive (with its OpenWrt install already on it) into a different laptop and needed that laptop's UEFI to actually recognize and boot from the transplanted drive. See [UEFI (Wikipedia)](https://en.wikipedia.org/wiki/UEFI).

## Secure Boot

A UEFI feature that only allows cryptographically signed bootloaders to run, rejecting anything unsigned at boot time. It's relevant to this bucket because the OpenWrt drive I moved out of the Acer ended up needing a Secure Boot workaround on its new host. See [[Homelab/Hardware/Lenovo-Thinkpad-P52s/Lenovo-thinkpad-p52s-glossary#Secure Boot|the canonical definition]] in the Lenovo ThinkPad P52s glossary.
