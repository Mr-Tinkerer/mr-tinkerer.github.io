---
title: Build and repairs
description: Getting the Dell Optiplex 5060 from a bare, RAM-less case to a working Proxmox host.
tags:
  - Homelab
  - Hardware
---
# RAM: buy ECC by mistake, then trade it

The cheapest used DDR4 RAM I found (Facebook Marketplace) turned out to be 2x8GB ECC RAM. [[Homelab/Hardware/Dell-Optiplex-5060/Dell-optiplex-5060-glossary#ECC RAM|ECC RAM]] needs explicit motherboard/CPU support, which this consumer board doesn't have — installing it produced a 3-beep BIOS POST error code (a RAM fault code). The seller had, in fact, flagged that it was ECC RAM before the sale; I missed the significance of that detail at the time.

I later traded the ECC kit, seller-to-seller, for an equivalent non-ECC 2x8GB DDR4 2400MHz kit, which the machine booted with normally.

# No mountable SSD, no video output

The 2.5" SSD I salvaged from the [[Homelab/Hardware/Acer-Aspire-3/Acer-aspire-3-overview|Acer laptop]] (from a previous, failed laptop-as-Proxmox-host attempt) was too small physically to sit in the Optiplex's 3.5" drive cage:

![[acer-ssd-too-small-for-optiplex-drive-cage.jpg]]
*The salvaged SSD does not fit the Optiplex's stock drive cage — no bracket for a 2.5" drive.*

I mounted it with duct tape across all four corners instead of a proper bracket, after removing the drive cage to make the work easier:

![[optiplex-ssd-mounted-with-duct-tape.png]]
*The SSD held down with duct tape in place of a missing 2.5"-to-3.5" mounting bracket.*

Separately, the Optiplex only outputs DisplayPort or VGA, while the monitor I had on hand is HDMI-only. I chained a borrowed DisplayPort→DVI-D adapter with a DVI-D→HDMI adapter to get a temporary picture for the OS install:

![[optiplex-booting-after-dp-to-hdmi-adapter-fix.jpg]]
*The Optiplex booting with output via the DisplayPort→DVI-D→HDMI adapter chain.*

# Result

With RAM and a display connected, I [[Homelab/Software/Proxmox/Proxmox-installation|installed Proxmox]] onto the taped-down SSD and the Optiplex became the home lab's hypervisor.
