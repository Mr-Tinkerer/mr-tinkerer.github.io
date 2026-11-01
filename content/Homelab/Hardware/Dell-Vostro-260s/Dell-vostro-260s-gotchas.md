---
title: Dell Vostro 260s Gotchas
description: Physical/firmware problems discovered after the NAS was moved into its permanent spot.
tags:
  - Homelab
  - Hardware
  - Gotchas
---

## Overview

This page documents physical/firmware problems discovered after the NAS was moved into its permanent spot.

## NAS Unreachable After Being Moved Into Place

After OpenMediaVault was installed and the machine (monitor disconnected) was moved to its permanent spot, its web UI became unreachable. Pulling the machine back out and reattaching a monitor showed it was stuck at the POST screen, waiting for a keypress to continue booting.

See [POST wait screenshot](Images/nas-post-screen-waiting-for-keypress.png) for the NAS stuck at POST, waiting for a keyboard press it will never get once deployed headless.

### Fix

The "wait for keypress on error/no keyboard" option was disabled in the BIOS, so the machine boots straight through with no keyboard attached.

See [headless boot screenshot](Images/nas-boots-headless-after-disabling-post-wait.png) for the NAS booting straight into OpenMediaVault with no keyboard attached, after the BIOS setting was disabled.

## BIOS Settings Resetting After a Long Power-Off

The fix above didn't stick permanently — after the machine was powered off for an extended period, the BIOS reverted to waiting for a keypress again. This is the classic symptom of a dead or missing CMOS battery failing to hold settings without wall power. Swapping the CMOS battery on the motherboard resolved it.
