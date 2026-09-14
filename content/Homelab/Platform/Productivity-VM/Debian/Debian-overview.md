---
title: Debian Overview
description: The Debian guest OS shared by every service on the Productivity VM.
tags:
  - Homelab
  - Debian
---
## Networking

The Debian installer didn't give me any IP configuration step, so it came up using DHCP by default. I set a static IP instead by editing `/etc/network/interfaces`:

```
# The primary network interface
allow-hotplug ens18
iface ens18 inet static
    address 10.10.0.2/28
    gateway 10.10.0.14
    dns-nameservers 10.10.0.14
```
