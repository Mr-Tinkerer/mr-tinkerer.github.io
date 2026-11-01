---
title: Acer Aspire 3 Overview
description: The laptop originally chosen to become the home lab's router, and why it was retired from that role before ever going into service.
tags:
  - Homelab
  - Hardware
---

## Overview

This page documents the Acer Aspire 3 A315-24PT laptop, originally chosen to become the home lab's router, and why it was retired from that role before ever going into service.

## What It Is

An Acer Aspire 3 A315-24PT laptop with an AMD Ryzen 5 7520U (2.8GHz), 8GB of RAM, and a 512GB NVMe SSD. It was picked as the router candidate because the dorm/apartment had no Ethernet port at all — every device had to reach the internet over the building's enterprise (WPA2-EAP) Wi-Fi, and running a dedicated router let the Wi-Fi login be kept in one place instead of configuring it on every server.

OpenWrt was installed onto this laptop's internal NVMe drive as the router OS (see the installation walkthrough for how).

## Why Retired from Router Duty

After OpenWrt was successfully installed to the internal drive, it became clear the laptop has no Ethernet port — only Wi-Fi. A router needs a wired LAN side to hand out to the rest of the home lab, so the Acer couldn't do the job as-is.

## The NVMe Transplant

Rather than reinstalling OpenWrt from scratch on a different machine, the Acer's NVMe drive (already containing a working OpenWrt install) was pulled and moved into the Lenovo ThinkPad P52s, which does have an Ethernet port. The drive wasn't soldered to the board, which made this possible — that was confirmed by opening the case with an iFixit "Jimmy" opening tool.

The internal battery was left disconnected during the teardown; it plays no further role since the Acer is no longer in service as a router.
