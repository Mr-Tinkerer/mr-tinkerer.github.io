---
title: Enterprise Wi-Fi (WPA2-EAP) authentication
description: Fixing a config-generation bug that broke PEAP/MSCHAPv2 authentication to a WPA2-Enterprise network.
tags:
  - Homelab
  - OpenWrt
---
# The symptom

With [[Homelab/Software/OpenWrt/Wifi-driver-installation|Wi-Fi drivers installed]], I found that connecting to the building's WPA2-EAP (PEAP / MSCHAPv2) network first required installing the `wpa_supplicant` package — without it, LuCI never even offers a Wi-Fi encryption option to select:

![[openwrt-missing-wpa-supplicant-package-warning.png]]
*LuCI warning that `wpa_supplicant` is missing before any Wi-Fi encryption method can be chosen.*

After I installed `wpa_supplicant` and configured PEAP with `EAP-MSCHAPv2` (the only inner-method option LuCI would accept — it rejects a bare `MSCHAPv2` selection as incompatible), every connection attempt failed immediately with a kernel-level error:

![[openwrt-ieee8021x-failed-kernel-error.png]]
*The `23=IEEE8021X_FAILED` error thrown on every connection attempt, regardless of EAP settings tried in the UI.*

I then connected the same router to a personal phone hotspot (WPA2-PSK), which worked immediately and could reach the internet — that isolated the problem to WPA2-Enterprise authentication specifically, not the Wi-Fi card or driver.

I spent about an hour searching online without finding anything useful, then spent roughly six hours working through it with Claude before landing on the actual root cause and fix below.

# Root cause

LuCI's config generator has a bug: when I select the inner authentication method as "EAP-MSCHAPv2" in the dropdown, it writes the literal string `EAP-MSCHAPV2` into the underlying `wpa_supplicant` config, instead of stripping the `EAP-` prefix as it's supposed to. The `wpa_supplicant` software only understands the bare method name `MSCHAPV2` — it doesn't recognize `EAP-MSCHAPV2` as valid, so it silently fails to complete that stage of authentication even with correct credentials.

# Fix

I set the underlying UCI value directly, bypassing the buggy dropdown-driven generator:

```
uci set wireless.@wifi-iface[1].auth='MSCHAPV2'
uci commit wireless
wifi reload
```

I adjusted `@wifi-iface[1]` to the actual index of the STA (client) interface — checked with `uci show wireless | grep auth` first.

![[openwrt-connected-to-enterprise-wifi-successfully.png]]
*A successful connection to the enterprise network after forcing the bare `MSCHAPV2` auth value.*

**Caveat:** I have to reapply this every time I change and save Wi-Fi settings from LuCI, since the web UI keeps rewriting the value back to the (invalid) `EAP-MSCHAPV2` form and will refuse to save at all until it's changed back to that value in the UI first:

![[openwrt-webui-blocks-changes-until-eap-mschapv2-set.png]]
*LuCI blocking further changes until the (invalid) `EAP-MSCHAPV2` value is put back in the UI field — the underlying config still needs the `uci` fix reapplied after saving.*
