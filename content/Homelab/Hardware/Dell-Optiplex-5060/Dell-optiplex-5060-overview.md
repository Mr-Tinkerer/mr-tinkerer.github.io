---
title: Dell Optiplex 5060 Overview
description: A free retired office PC that became the home lab's Proxmox hypervisor.
tags:
  - Homelab
  - Hardware
---
# What it is

A Dell Optiplex 5060 that I got free from a college IT department clearing out its retired-PC "graveyard." It has a proprietary (non-standard-form-factor) motherboard, so I can't move its board into a different case — unlike the [[Homelab/Hardware/Dell-Vostro-260s/Dell-vostro-260s-overview|Dell Vostro 260s]], which uses a normal ATX board. It supports DDR4 RAM and has a better CPU than the Vostro, so I designated it the [[Homelab/Software/Proxmox/Proxmox-installation|Proxmox]] hypervisor for the home lab.

# Condition as received

The PC arrived with a Seagate 500GB hard drive but **no RAM sticks** — the donor department's policy required removing any DDR4 RAM or SSD/NVMe storage before giving a machine away, due to a RAM shortage at the time. This meant I couldn't boot the machine until I sourced replacement RAM.

![[optiplex-missing-ram-sticks-as-received.png]]
*The Optiplex as received — no RAM installed, per the donor's take-back policy.*

Its CPU is an Intel Core i7-8700, later relevant to the [[Homelab/Logical/Virtual-machines/Vm-allocation|Local AI VM]]'s CPU-only inference performance since that VM runs on this host.

See [[Homelab/Hardware/Dell-Optiplex-5060/Build-and-repairs|Build and repairs]] for how I brought it up to a working state.
