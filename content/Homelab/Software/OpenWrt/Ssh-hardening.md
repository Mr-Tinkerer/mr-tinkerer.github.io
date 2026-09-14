---
title: SSH hardening
description: Key-based auth, a non-default port, and a per-host SSH config entry for the OpenWrt router.
tags:
  - Homelab
  - OpenWrt
---
# Key setup

I generated an ed25519 keypair on the management machine with `ssh-keygen -t ed25519 -f ~/.ssh/OpenWrt`, then pasted the public key into `System → Administration → SSH-Keys` in LuCI to authorize it for the root account.

# Non-default port and disabled password auth

Under `System → Administration → SSH Access`:
- I moved SSH off port 22 to a randomly-chosen high port, mainly so casual/automated bot scans don't immediately see SSH running on the default port. This is a minor obscurity measure, not a real access control — the router isn't exposed to the public internet regardless.
- I disabled password authentication entirely, leaving key-based auth as the only way in. This is the control that actually matters, since it removes brute-forceable credentials as an attack surface.
- I also restricted access to the LAN interface only.

# Client-side config

To avoid re-typing the host/port/key path every time, I added an entry to the management machine's `~/.ssh/config`:

```ssh-config
Host OpenWrt
	HostName 192.168.1.1
	user root
	IdentityFile ~/.ssh/OpenWrt
	Port 35478
```

With this in place, `ssh OpenWrt` connects directly. Note the `HostName` here reflects the router's original `192.168.1.1` address, before the home lab's addressing scheme was applied — see [[Homelab/Logical/Home-network/Addressing-plan|the Home-network addressing plan]] for the router's current LAN address.
