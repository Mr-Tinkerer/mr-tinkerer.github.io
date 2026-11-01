---
title: OpenWrt SSH Hardening
description: Key-based auth, a non-default port, and a per-host SSH config entry for the OpenWrt router.
tags:
  - Homelab
  - OpenWrt
---

## Overview

This page documents key-based auth, a non-default port, and a per-host SSH config entry for the OpenWrt router.

## Key Setup

An ed25519 keypair was generated on the management machine with `ssh-keygen -t ed25519 -f ~/.ssh/OpenWrt`, then the public key was pasted into `System → Administration → SSH-Keys` in LuCI to authorize it for the root account.

## Non-Default Port and Disabled Password Auth

Under `System → Administration → SSH Access`:

- **Port change** — SSH was moved off port 22 to a randomly-chosen high port, mainly so casual/automated bot scans don't immediately see SSH running on the default port. This is a minor obscurity measure, not a real access control — the router isn't exposed to the public internet regardless
- **Password auth disabled** — Password authentication was disabled entirely, leaving key-based auth as the only way in. This is the control that actually matters, since it removes brute-forceable credentials as an attack surface
- **LAN-only access** — Access was restricted to the LAN interface only

## Client-Side Config

To avoid re-typing the host/port/key path every time, an entry was added to the management machine's `~/.ssh/config`:

```ssh-config
Host OpenWrt
    HostName 192.168.1.1
    user root
    IdentityFile ~/.ssh/OpenWrt
    Port 35478
```

With this in place, `ssh OpenWrt` connects directly. Note the `HostName` reflects the router's original `192.168.1.1` address, before the home lab's addressing scheme was applied — see the Home-network addressing plan for the router's current LAN address.
