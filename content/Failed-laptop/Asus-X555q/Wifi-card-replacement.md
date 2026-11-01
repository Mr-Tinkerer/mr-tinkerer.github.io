---
title: Diagnosing and Replacing the Wi-Fi Card
description: How the Asus X555q's onboard Wi-Fi card was diagnosed as faulty across three OSes and replaced with a USB dongle.
tags:
  - Asus-X555q
  - Hardware
  - Wifi
---

## Overview

This page documents how the Asus X555q's onboard Wi-Fi card was diagnosed as faulty across three operating systems and replaced with a USB dongle.

## Symptoms

The laptop (Asus X555q, AMD A12-9720P APU, 16GB RAM, 1TB SSD) connects to Wi-Fi successfully but loses the connection within seconds — pings to `google.com` stop responding after a few packets, and the network eventually drops from the list of visible networks entirely.

## Ruling out Software

The same failure was reproduced across three different operating systems on the same hardware:

- Ubuntu 24.04
- EndeavourOS (Arch Linux, latest packages)
- Windows 10 (the OS the laptop originally shipped with)

Since the symptom persisted across OSes with completely different Wi-Fi drivers, the OS-agnostic test pointed to a hardware fault rather than a driver bug — the fix (replace the card) doesn't depend on which OS is installed.

## Opening the Laptop

A teardown video was followed for disassembly. The order of removal was: keyboard (unclipped with a plastic pry tool, ribbon cable disconnected last to avoid tearing it), battery, CD drive, then the hard drive assembly (which required disconnecting a ribbon cable and unscrewing a bridge component before it would come free), and finally the motherboard itself — held in by one easy-to-miss screw in addition to the visible ones.

See [labeled internals](Images/asus-x555q-internals-labeled.png) for labeled internals after removing the keyboard, battery, CD drive, and hard drive.

Flipping the motherboard over confirmed the Wi-Fi card's location and let it be identified directly from the part markings on the card:

See [Wi-Fi card closeup](Images/asus-x555q-wifi-card-closeup.png) for the onboard mini-PCIe Wi-Fi card, later confirmed faulty.

## Replacement Decision

A same-model replacement mini-PCIe card was found online, but since no listing would ship in a reasonable time, the faulty card was removed entirely and replaced with a USB Wi-Fi dongle that had already been confirmed to work with Linux from prior use. All other components were reassembled in reverse order.

## Gotchas

- When a symptom reproduces identically across unrelated OSes/drivers on the same physical hardware, stop chasing driver/config fixes and treat it as a hardware fault
- Don't skip a screw before forcing a component out — the motherboard resisted removal only because one extra screw (not visible in the reference teardown video) was still in place
