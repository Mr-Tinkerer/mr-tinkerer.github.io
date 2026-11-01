---
title: Dell Optiplex 5060 Build and Repairs
description: Getting the Dell Optiplex 5060 from a bare, RAM-less case to a working Proxmox host.
tags:
  - Homelab
  - Hardware
---

## Overview

This page documents getting the Dell Optiplex 5060 from a bare, RAM-less case to a working Proxmox host.

## RAM: Buy ECC by Mistake, Then Trade It

The cheapest used DDR4 RAM found (Facebook Marketplace) turned out to be 2x8GB ECC RAM. ECC RAM needs explicit motherboard/CPU support, which this consumer board doesn't have — installing it produced a 3-beep BIOS POST error code (a RAM fault code). The seller had, in fact, flagged that it was ECC RAM before the sale; the significance of that detail was missed at the time.

The ECC kit was later traded, seller-to-seller, for an equivalent non-ECC 2x8GB DDR4 2400MHz kit, which the machine booted with normally.

## No Mountable SSD, No Video Output

The 2.5" SSD salvaged from the Acer laptop (from a previous, failed laptop-as-Proxmox-host attempt) was too small physically to sit in the Optiplex's 3.5" drive cage:

See [SSD too small screenshot](Images/acer-ssd-too-small-for-optiplex-drive-cage.jpg) for the salvaged SSD not fitting the Optiplex's stock drive cage — no bracket for a 2.5" drive.

It was mounted with duct tape across all four corners instead of a proper bracket, after removing the drive cage to make the work easier:

See [duct tape mount screenshot](Images/optiplex-ssd-mounted-with-duct-tape.png) for the SSD held down with duct tape in place of a missing 2.5"-to-3.5" mounting bracket.

Separately, the Optiplex only outputs DisplayPort or VGA, while the monitor on hand was HDMI-only. A borrowed DisplayPort→DVI-D adapter chained with a DVI-D→HDMI adapter got a temporary picture for the OS install:

See [adapter chain screenshot](Images/optiplex-booting-after-dp-to-hdmi-adapter-fix.jpg) for the Optiplex booting with output via the DisplayPort→DVI-D→HDMI adapter chain.

## Result

With RAM and a display connected, Proxmox was installed onto the taped-down SSD and the Optiplex became the home lab's hypervisor.
