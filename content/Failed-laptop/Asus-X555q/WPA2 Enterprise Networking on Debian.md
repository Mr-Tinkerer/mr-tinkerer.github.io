---
title: WPA2 Enterprise Networking on Debian
description: Connecting to a WPA2 Enterprise network with wpa_supplicant on a headless Debian/Proxmox install, and why static IPs don't work on this network.
tags:
  - Asus-X555q
  - Debian
  - Wifi
  - Networking
---

## Overview

This page covers connecting to a WPA2 Enterprise network with wpa_supplicant on a headless Debian/Proxmox install, and why static IPs don't work on this network.

## Why a Desktop Environment Matters at First

GNOME's network UI has no fields for WPA2 Enterprise (username/password/PEAP), so a GNOME-based live USB (e.g., Ubuntu desktop) couldn't join the enterprise network at all. KDE does expose these fields.

See [KDE Wi-Fi fields screenshot](Images/kde-wpa2-enterprise-connect-fields.png) for KDE's Wi-Fi connection dialog exposing the WPA2 Enterprise fields (security type, PEAP, inner authentication, identity/password) that GNOME's equivalent dialog lacks.

This was only needed to get initial connectivity for OS installation/testing. The actual server doesn't run a desktop environment.

## Connecting Headless with wpa_supplicant

Debian (and Proxmox, which is Debian-based) uses ifupdown rather than NetworkManager by default. NetworkManager and ifupdown cannot manage the same Wi-Fi interface simultaneously — only one network tool can hold an interface at a time, so ifupdown had to be disabled before NetworkManager (or vice versa) could see the adapter.

### wpa_supplicant Configuration

Many templates found online for the wpa_supplicant entry were tried and none connected. A working config was generated and added to `/etc/wpa_supplicant/wpa_supplicant.conf`:

```toml
ctrl_interface=DIR=/run/wpa_supplicant GROUP=netdev
update_config=1

network={
        ssid="SSID"
        key_mgmt=WPA-EAP
        eap=PEAP
        identity="USERNAME"
        password="PASSWORD"
        phase2="auth=MSCHAPV2"
}
```

That config is attached to the interface in `/etc/network/interfaces`:

```
auto wlx08bfb84f0c66
iface wlx08bfb84f0c66 inet dhcp
        gateway 10.77.160.1
        wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```

See [Reference: Debian Wi-Fi setup](https://wiki.debian.org/WiFi/HowToUse#A.2Fetc.2Fnetwork.2Finterfaces_and_wpa_supplicant) and [Reference: Gentoo wpa_supplicant guide](https://wiki.gentoo.org/wiki/Wpa_supplicant#WPA2_with_wpa_supplicant) for the general templates originally tried adapting before landing on the config above.

## Static IPs Don't Work on This Network

The standard Debian static-IP procedure was followed and it didn't work — the interface only comes up when configured for DHCP. This was reproduced across every guide and reboot tried.

### Root Cause

The dorm's network was blocking static IP assignment outright. Per the OS-agnostic test: since this is a dorm network not controlled, and the same failure occurred regardless of the OS or static-IP method used, that explanation fits — it's the only thing that accounts for why no guide worked.

As a result, the laptop was left on DHCP. See [[Failed-laptop/Asus-X555q/Installing Proxmox on a Wi-Fi-Only Machine#Tracking a DHCP-only IP|Proxmox-installation]] for how the server's hostname/IP is tracked despite this.

## Gotchas

- **Network tool conflict** — If a Wi-Fi adapter that worked fine under NetworkManager stops being visible after switching to ifupdown (or vice versa), that's expected — disable one before expecting the other to manage the interface
- **Static IP failure** — If a static IP consistently fails to come up while DHCP for the same interface consistently works, and the network hardware isn't controlled, suspect the network is blocking static assignment before assuming a local config error
