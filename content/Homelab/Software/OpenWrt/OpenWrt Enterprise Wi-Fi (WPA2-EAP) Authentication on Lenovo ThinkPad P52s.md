---
title: OpenWrt Enterprise Wi-Fi (WPA2-EAP) Authentication on Lenovo ThinkPad P52s
description: Fixing a LuCI configuration generator bug that broke PEAP/MSCHAPv2 authentication to WPA2-Enterprise networks on the router.
tags:
  - Homelab
  - OpenWrt
  - Networking
---

## Overview

This guide covers configuring WPA2-EAP (PEAP/MSCHAPv2) enterprise Wi-Fi authentication on OpenWrt, addressing a LuCI configuration generator bug that inserts an invalid authentication method prefix.

## Observed Issue

### Prerequisites

Wi-Fi drivers must be installed before configuring enterprise authentication.

### Initial Requirement

The `wpa_supplicant` package is required for LuCI to present Wi-Fi encryption selection options. Without installation, the encryption selection dropdown is unavailable.

See [missing package warning](Images/openwrt-missing-wpa-supplicant-package-warning.png) for the LuCI warning before Wi-Fi encryption method selection.

### Authentication Failure

After installing `wpa_supplicant` and configuring PEAP with `EAP-MSCHAPv2` (the only inner-method option LuCI accepts; bare `MSCHAPv2` is rejected as incompatible), every connection attempt fails immediately with a kernel-level error.

See [IEEE8021X_FAILED error](Images/openwrt-ieee8021x-failed-kernel-error.png) for the `23=IEEE8021X_FAILED` error on every connection attempt.

### Issue Isolation

Connecting the same router to a personal phone hotspot (WPA2-PSK) succeeds immediately with internet access, isolating the problem to WPA2-Enterprise authentication specifically, not the Wi-Fi card or driver stack.

## Root Cause Analysis

### LuCI Configuration Generator Bug

The LuCI configuration generator inserts the literal string `EAP-MSCHAPV2` into the underlying `wpa_supplicant` configuration when the `EAP-MSCHAPv2` option is selected, instead of stripping the `EAP-` prefix as intended.

### wpa_supplicant Requirement

The `wpa_supplicant` daemon requires the bare, canonical method identifier `MSCHAPV2`. The `EAP-` prefix is not recognized as valid, causing the authentication stage to fail silently despite correct credentials.

## Solution

### Implementation Protocol

Override the flawed LuCI configuration generator by directly modifying the underlying system settings using the UCI command-line interface.

### Steps

1. **Identify STA Interface:**
   ```bash
   uci show wireless | grep auth
   ```
   Determine the correct index (`<index>`) for the client (STA) interface.

2. **Inject Correct Auth Value:**
   ```bash
   uci set wireless.@wifi-iface[<index>].auth='MSCHAPV2'
   ```
   Apply the bare method name directly.

3. **Commit and Reload:**
   ```bash
   uci commit wireless
   wifi reload
   ```

### Interface Index Adjustment

Adjust `@wifi-iface[1]` to match the actual STA (client) interface index, verified via `uci show wireless | grep auth`.

See [successful connection](Images/openwrt-connected-to-enterprise-wifi-successfully.png) for confirmation after forcing the bare `MSCHAPV2` auth value.

## Caveat: Web UI State Interference

### Problem

The LuCI web interface maintains persistent state that enforces the invalid `EAP-MSCHAPV2` value. Modifications made within the LuCI GUI revert the underlying configuration.

### Behavior

- Saving Wi-Fi settings through LuCI rewrites the value back to `EAP-MSCHAPV2`
- The web interface refuses to save until the invalid value is re-entered in the UI first

### Mitigation

Reapply the `uci` command sequence above whenever Wi-Fi settings are modified or saved through the graphical interface.

See [UI blocking screenshot](Images/openwrt-webui-blocks-changes-until-eap-mschapv2-set.png) for LuCI blocking further changes until the invalid value is restored in the UI field.
