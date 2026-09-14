---
title: Connecting to a WPA2 Enterprise Network (No Desktop Environment)
description: Getting a headless Debian/Proxmox install onto a WPA2 Enterprise Wi-Fi network via wpa_supplicant, and why static IPs don't work on this network.
tags:
  - Asus-X555q
  - Debian
  - Wifi
  - Networking
---
## Why a desktop environment matters at first

[[Asus-x555q-glossary#GNOME|GNOME]]'s network UI has no fields for [[Asus-x555q-glossary#WPA2 Enterprise|WPA2 Enterprise]] (username/password/PEAP), so a GNOME-based live USB (e.g. Ubuntu desktop) couldn't join the enterprise network at all. [[Asus-x555q-glossary#KDE|KDE]] does expose these fields:

![[kde-wpa2-enterprise-connect-fields.png]]
*KDE's Wi-Fi connection dialog exposes the WPA2 Enterprise fields (security type, PEAP, inner authentication, identity/password) that GNOME's equivalent dialog lacks.*

I only needed this to get initial connectivity for OS installation/testing. The actual server doesn't run a desktop environment.

## Connecting headless with wpa_supplicant

Debian (and Proxmox, which is Debian-based) uses [[Asus-x555q-glossary#ifupdown|`ifupdown`]] rather than `NetworkManager` by default. `NetworkManager` and `ifupdown` cannot manage the same Wi-Fi interface simultaneously — only one network tool can hold an interface at a time, so I had to disable `ifupdown` before `NetworkManager` (or vice versa) could see the adapter.

I tried many templates I found online for the `wpa_supplicant` entry and none of them connected. I eventually got a working config by asking Google Gemini to generate one for me, added to `/etc/wpa_supplicant/wpa_supplicant.conf`:

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

See [Reference: Debian Wi-Fi setup](https://wiki.debian.org/WiFi/HowToUse#A.2Fetc.2Fnetwork.2Finterfaces_and_wpa_supplicant) and [Reference: Gentoo wpa_supplicant guide](https://wiki.gentoo.org/wiki/Wpa_supplicant#WPA2_with_wpa_supplicant) for the general templates I originally tried adapting before landing on the Gemini-generated config above.

## Static IPs don't work on this network

I followed the [standard Debian static-IP procedure](https://wiki.debian.org/NetworkConfiguration#Configuring_the_interface_manually) and it didn't work — the interface only comes up when configured for DHCP. I reproduced this across every guide and reboot I tried.

I was stuck until I asked Google Gemini for help, and it suggested that the dorm's network could be blocking static IP assignment outright. Per the [[Asus-x555q-glossary#OS-agnostic test|OS-agnostic test]]: since this is a dorm network I don't control the router/firewall for, and the same failure occurred regardless of the OS or static-IP method I used, that explanation fit — it's the only thing that accounts for why no guide worked. I left the laptop on DHCP as a result — see [[Failed-laptop/Asus-X555q/Proxmox-installation#Tracking a DHCP-only IP|Proxmox-installation]] for how I track the server's hostname/IP despite this.

## Gotchas

- If a Wi-Fi adapter that worked fine under `NetworkManager` stops being visible after you switch to `ifupdown` (or vice versa), that's expected — disable one before expecting the other to manage the interface.
- If a static IP consistently fails to come up while DHCP for the same interface consistently works, and you don't control the network hardware, suspect the network is blocking static assignment before assuming a local config error.
