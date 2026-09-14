---
title: Diagnosing and Replacing the Wi-Fi Card
description: How the Asus X555q's onboard Wi-Fi card was diagnosed as faulty across three OSes and replaced with a USB dongle.
tags:
  - Asus-X555q
  - Hardware
  - Wifi
---
w## Symptoms

My laptop (Asus X555q, AMD A12-9720P APU, 16GB RAM, 1TB SSD) connects to Wi-Fi successfully but loses the connection within seconds — pings to `google.com` stop responding after a few packets, and the network eventually drops from the list of visible networks entirely.

## Ruling out software

I reproduced the same failure across three different operating systems on the same hardware:

- Ubuntu 24.04
- EndeavourOS (Arch Linux, latest packages)
- Windows 10 (the OS the laptop originally shipped with)

Since the symptom persisted across OSes with completely different Wi-Fi drivers, the [[Asus-x555q-glossary#OS-agnostic test|OS-agnostic test]] pointed me to a hardware fault rather than a driver bug — the fix (replace the card) doesn't depend on which OS is installed.

## Opening the laptop

I followed this [iFixit-style teardown video](https://www.youtube.com/watch?v=kpkZ6dwhnB0) for disassembly. My order of removal: keyboard (unclipped with a plastic pry tool, ribbon cable disconnected last to avoid tearing it), battery, CD drive, then the hard drive assembly (which required disconnecting a ribbon cable and unscrewing a bridge component before it would come free), and finally the motherboard itself — held in by one easy-to-miss screw in addition to the visible ones.

![[asus-x555q-internals-labeled.png]]
*Labeled internals after removing the keyboard, battery, CD drive, and hard drive.*

Flipping the motherboard over confirmed the Wi-Fi card's location and let me identify it directly from the part markings on the card:

![[asus-x555q-wifi-card-closeup.png]]
*The onboard mini-PCIe Wi-Fi card, later confirmed faulty.*

## Replacement decision

I found a same-model replacement mini-PCIe card [online](https://www.amazon.ca/MPE-AXE3000H-AX210-AX3000Mbps-Bluetooth-Adapter/dp/B0DP9YKVRL), but since no listing would ship in a reasonable time, I removed the faulty card entirely and replaced it with a [USB Wi-Fi dongle](https://www.asus.com/ca-en/networking-iot-servers/adapters/all-series/usb-ac53-nano/) I'd already confirmed worked with Linux from prior use. I reassembled all other components in reverse order.

## Gotchas

- When a symptom reproduces identically across unrelated OSes/drivers on the same physical hardware, stop chasing driver/config fixes and treat it as a hardware fault.
- Don't skip a screw before forcing a component out — the motherboard resisted removal only because one extra screw (not visible in the reference teardown video) was still in place.
