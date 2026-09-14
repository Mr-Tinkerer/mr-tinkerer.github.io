---
title: Installing Proxmox on a Wi-Fi-Only Machine
description: Getting Proxmox VE running over a USB Wi-Fi dongle instead of the Ethernet connection it expects, including NAT, routing, and storage layout fixes.
tags:
  - Asus-X555q
  - Proxmox
  - Networking
---
## Why the standard installer doesn't work here

The Proxmox VE installer assumes an Ethernet connection and has no path to configure Wi-Fi during setup. I installed [Proxmox on top of Debian 13](https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_13_Trixie) instead of using the ISO to avoid that assumption, since I could configure Debian with `wpa_supplicant` first (see [[Wpa2-enterprise-networking|WPA2 Enterprise Networking]]) and layer Proxmox on afterward.

A Proxmox-on-Debian install only ships the `standard system utilities` package set — no `wpa_supplicant` — so I had to add the package manually with no internet access to fetch it directly:

1. Build a throwaway Debian VM and repackage the installed `wpa_supplicant` into a portable `.deb` with `dpkg-repack wpasupplicant`.
2. Copy the resulting `.deb` to the target machine and install it with `dpkg -i wpasupplicant_2.10-24_amd.deb`.
3. Configure `/etc/wpa_supplicant/wpa_supplicant.conf` and `/etc/network/interfaces` as described in [[Wpa2-enterprise-networking|WPA2 Enterprise Networking]].

After connecting, the interface received an IP over DHCP but couldn't reach the internet — `ip route` showed no gateway had been set. The DHCP lease gave me an IP but not a working default route; manually assigning the gateway fixed connectivity.

## Tracking a DHCP-only IP

Since the network blocks static IPs (see [[Wpa2-enterprise-networking|WPA2 Enterprise Networking]]), I had to keep Proxmox's requirement that `/etc/hosts` match its current IP in sync automatically. I used a community DHCP hook script (posted in the [Proxmox forum](https://forum.proxmox.com/threads/set-dhcp-on-proxmox.168358/#post-782918)) that updates `/etc/hosts` on every lease renewal.

I extended the same hook to also rewrite the login splash screen (`/etc/issue`) with the current IP. I don't know `sed`/regex syntax well, so I had Google Gemini generate the command for me:

```bash
sed -i "s/https:\/\/[0-9]\{1,3\}\(\.[0-9]\{1,3\}\)\{3\}/https:\/\/$new_ip_address/" /etc/issue
```

![[proxmox-splash-screen-dynamic-ip.png]]
*The login splash screen showing the current Web UI address, kept accurate across DHCP lease changes.*

## NAT, bridging, and routing for VM traffic

VMs attach to a Proxmox network bridge (`vmbr0`), but the dorm network's firewall blocks bridged traffic outright — VMs on the bridge couldn't reach the internet even with a correctly configured bridge. I was stuck on this until I asked Google Gemini, which suggested the dorm's firewall was blocking bridged traffic specifically. That led me to use a NAT interface instead of a bridge, following [this guide](https://cloudspinx.com/creating-private-network-bridge-on-proxmox-ve-with-nat/): create a private interface for the VMs, then have the Wi-Fi interface masquerade and IP-forward for it.

For port forwarding into a VM/container, I asked Gemini how to set it up, and used the `iptables` DNAT rules it gave me, attached to the Wi-Fi interface's up/down hooks:

```bash
post-up   iptables -t nat -A PREROUTING -i wlx08bfb84f0c66 -p tcp --dport 8080 -j DNAT --to-destination 192.168.50.2:90
post-down iptables -t nat -D PREROUTING -i wlx08bfb84f0c66 -p tcp --dport 8080 -j DNAT --to-destination 192.168.50.2:90
```

`post-up`/`post-down` run when the interface goes up or down. `-t nat` selects the NAT table; `-A`/`-D` add or delete the rule. `-i wlx08bfb84f0c66` is the Wi-Fi dongle's interface name. `-j DNAT --to-destination 192.168.50.2:90` redirects incoming traffic on the dongle's port 8080 to port 90 on the internal VM.

### Routing conflict after adding Ethernet

Once I added a wired connection for stable Web UI access (see [[Lid-and-power-management|Lid and Power Management]] for why Wi-Fi remains the VM/internet path), Proxmox silently changed the default route to go out through the new Ethernet bridge (`vmbr0`) instead of the Wi-Fi dongle, breaking outbound VM traffic. `ip route` confirmed it:

```
default via 192.168.100.1 dev vmbr0 proto kernel onlink
10.77.160.0/19 dev wlx08bfb84f0c66 proto kernel scope link src 10.77.160.104
192.168.50.0/24 dev vmbr1 proto kernel scope link src 192.168.50.1
192.168.100.0/24 dev vmbr0 proto kernel scope link src 192.168.100.2
```

I fixed it by forcing the default route back onto the Wi-Fi interface whenever `vmbr0` comes up, and restoring the Ethernet route when it goes down, in `/etc/network/interfaces`:

```
post-up ip route delete 0.0.0.0/0
post-up ip route add 0.0.0.0/0 via 10.77.160.1 dev wlx08bfb84f0c66

post-down ip route delete 0.0.0.0/0
post-down ip route add 0.0.0.0/0 via 192.168.100.1 dev vmbr0
```

## Merging the local and local-lvm storage pools

By default Proxmox splits disk space between two storage pools, `local` and `local-lvm`. On my install the split left `local` too small to be useful on its own. Following [this guide](https://blog.gnomeitsolutions.com/delete-local-lvm-proxmox/), I tested the fix first in a Proxmox VM before applying it to the physical machine:

1. Remove the `local-lvm` storage entry in Proxmox and let `local` handle all storage content types.
2. Delete the LVM volume: `lvremove /dev/pve/data`.
3. Grow the `root` logical volume into the freed space: `lvresize -l +100%FREE /dev/pve/root`.
4. Resize the filesystem to match: `resize2fs /dev/mapper/pve-root`.

I confirmed disk images, containers, and backups/snapshots worked on the merged `local` pool before repeating the same steps on the actual laptop.

## Gotchas

- If a bridged VM can reach the bridge but not the internet, and you don't control the upstream network, try NAT instead of a bridge before assuming the bridge config is wrong.
- Adding a second network interface (e.g. Ethernet) to a Proxmox box can silently change the default route. Recheck `ip route` after any interface change if a route that previously worked stops working.
