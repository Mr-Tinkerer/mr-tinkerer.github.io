---
title: Installing Proxmox on a Wi-Fi-Only Machine
description: Getting Proxmox VE running over a USB Wi-Fi dongle instead of the Ethernet connection it expects, including NAT, routing, and storage layout fixes.
tags:
  - Asus-X555q
  - Proxmox
  - Networking
---

## Overview

This page covers getting Proxmox VE running over a USB Wi-Fi dongle instead of the Ethernet connection it expects, including NAT, routing, and storage layout fixes.

## Why the Standard Installer Doesn't Work Here

The Proxmox VE installer assumes an Ethernet connection and has no path to configure Wi-Fi during setup. Proxmox was installed on top of Debian 13 instead of using the ISO to avoid that assumption, since Debian could be configured with wpa_supplicant first and Proxmox layered on afterward.

### Package Installation

A Proxmox-on-Debian install only ships the `standard system utilities` package set — no wpa_supplicant — so the package had to be added manually with no internet access to fetch it directly:

1. Build a throwaway Debian VM and repackage the installed wpa_supplicant into a portable .deb with `dpkg-repack wpasupplicant`
2. Copy the resulting .deb to the target machine and install it with `dpkg -i wpasupplicant_2.10-24_amd.deb`
3. Configure `/etc/wpa_supplicant/wpa_supplicant.conf` and `/etc/network/interfaces` as described in WPA2 Enterprise Networking

### Connectivity Issue

After connecting, the interface received an IP over DHCP but couldn't reach the internet — `ip route` showed no gateway had been set. The DHCP lease gave an IP but not a working default route; manually assigning the gateway fixed connectivity.

## Tracking a DHCP-Only IP

Since the network blocks static IPs, Proxmox's requirement that `/etc/hosts` match its current IP needed to be kept in sync automatically. A community DHCP hook script (posted in the Proxmox forum) was used that updates `/etc/hosts` on every lease renewal.

### Splash Screen Update

The same hook extended to also rewrite the login splash screen (`/etc/issue`) with the current IP:

```bash
sed -i "s|https://$new_ip_address|" /etc/issue
```

See [splash screen screenshot](Images/proxmox-splash-screen-dynamic-ip.png) for the login splash screen showing the current Web UI address, kept accurate across DHCP lease changes.

## NAT, Bridging, and Routing for VM Traffic

VMs attach to a Proxmox network bridge (`vmbr0`), but the dorm network's firewall blocks bridged traffic outright — VMs on the bridge couldn't reach the internet even with a correctly configured bridge. A NAT interface was used instead of a bridge, following a guide: create a private interface for the VMs, then have the Wi-Fi interface masquerade and IP-forward for it.

### Port Forwarding

For port forwarding into a VM/container, iptables DNAT rules were used, attached to the Wi-Fi interface's up/down hooks:

```bash
post-up   iptables -t nat -A PREROUTING -i wlx08bfb84f0c66 -p tcp --dport 8080 -j DNAT --to-destination 192.168.50.2:90
post-down iptables -t nat -D PREROUTING -i wlx08bfb84f0c66 -p tcp --dport 8080 -j DNAT --to-destination 192.168.50.2:90
```

- `post-up`/`post-down` run when the interface goes up or down
- `-t nat` selects the NAT table; `-A`/`-D` add or delete the rule
- `-i wlx08bfb84f0c66` is the Wi-Fi dongle's interface name
- `-j DNAT --to-destination 192.168.50.2:90` redirects incoming traffic on the dongle's port 8080 to port 90 on the internal VM

### Routing Conflict After Adding Ethernet

Once a wired connection was added for stable Web UI access, Proxmox silently changed the default route to go out through the new Ethernet bridge (`vmbr0`) instead of the Wi-Fi dongle, breaking outbound VM traffic. `ip route` confirmed it:

```
default via 192.168.100.1 dev vmbr0 proto kernel onlink
10.77.160.0/19 dev wlx08bfb84f0c66 proto kernel scope link src 10.77.160.104
192.168.50.0/24 dev vmbr1 proto kernel scope link src 192.168.50.1
192.168.100.0/24 dev vmbr0 proto kernel scope link src 192.168.100.2
```

The fix forced the default route back onto the Wi-Fi interface whenever `vmbr0` comes up, and restoring the Ethernet route when it goes down, in `/etc/network/interfaces`:

```
post-up ip route delete 0.0.0.0/0
post-up ip route add 0.0.0.0/0 via 10.77.160.1 dev wlx08bfb84f0c66

post-down ip route delete 0.0.0.0/0
post-down ip route add 0.0.0.0/0 via 192.168.100.1 dev vmbr0
```

## Merging the Local and Local-LVM Storage Pools

By default Proxmox splits disk space between two storage pools, `local` and `local-lvm`. On the install the split left `local` too small to be useful on its own. Following a guide, the fix was tested first in a Proxmox VM before applying it to the physical machine:

1. Remove the `local-lvm` storage entry in Proxmox and let `local` handle all storage content types
2. Delete the LVM volume: `lvremove /dev/pve/data`
3. Grow the `root` logical volume into the freed space: `lvresize -l +100%FREE /dev/pve/root`
4. Resize the filesystem to match: `resize2fs /dev/mapper/pve-root`

Disk images, containers, and backups/snapshots were confirmed to work on the merged `local` pool before repeating the same steps on the actual laptop.

## Gotchas

- **Bridged VM without internet** — If a bridged VM can reach the bridge but not the internet, and the upstream network isn't controlled, try NAT instead of a bridge before assuming the bridge config is wrong
- **Default route change** — Adding a second network interface (e.g., Ethernet) to a Proxmox box can silently change the default route. Recheck `ip route` after any interface change if a route that previously worked stops working
