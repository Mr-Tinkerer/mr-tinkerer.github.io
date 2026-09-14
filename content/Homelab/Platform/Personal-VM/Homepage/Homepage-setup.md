---
title: Homepage Setup
description: Standing up a Homepage dashboard on the Personal VM to avoid typing full home lab hostnames.
tags:
  - Homelab
  - Homepage
---
# Why

The home lab's custom, unregistered TLD (see [[Homelab/Logical/Home-network/Dns-naming-scheme|the DNS naming scheme]]) means browsers often treat a bare hostname as a search query instead of a URL unless `https://` is typed explicitly. [Homepage](https://gethomepage.dev/) provides a single bookmarkable dashboard linking to every home lab service instead.

# Running it

Homepage runs as a [[Homelab/Logical/Containerization/Tool-choices|Podman]] quadlet-managed container on the [[Homelab/Logical/Virtual-machines/Vm-allocation|Personal VM]] (openSUSE), reachable at `home.uhhhhh` through [[Homelab/Software/Nginx/Reverse-proxy-setup|Nginx]] (see [home.conf](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Nginx/conf.d/home.conf)). The maintained quadlet and config volume are kept in the infrastructure repo: [homepage.container](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Personal%20VM/Homepage/Quadlet/homepage.container), [homepage.network](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Personal%20VM/Homepage/Quadlet/homepage.network), and its [config directory](https://github.com/Mr-Tinkerer/Project-Dump/tree/main/Homelab/Personal%20VM/Homepage/Volume).

# Gotchas

Getting the container reachable and writable at all took three separate fixes. Two of them are openSUSE OS-level defaults, not anything specific to Homepage — see [[Homelab/Platform/Personal-VM/Opensuse/Opensuse-gotchas|openSUSE: Gotchas]] for the `firewalld` and SELinux fixes in full. The third was specific to this service:

- **Config changes weren't taking effect.** Editing Homepage's mounted settings file had no effect until I gave the container its own dedicated volume, rather than relying on defaults baked into the image.

# Integrations

I connected Homepage to several other home lab services via their own integrations: [[Homelab/Platform/Productivity-VM/Nextcloud/Nextcloud-installation|Nextcloud]] (via a long-lived API token), [[Homelab/Software/OpenWrt/Openwrt-overview|OpenWrt]] and [[Homelab/Software/OpenMediaVault/Openmediavault-installation|OpenMediaVault]] (via login credentials), [[Homelab/Software/Tailscale/Tailscale-setup|Tailscale]], and [[Homelab/Software/Proxmox/Proxmox-installation|Proxmox]] (via an API token — Proxmox's widget did not display data correctly at time of writing). Custom service icons come from the [homarr-labs/dashboard-icons](https://github.com/homarr-labs/dashboard-icons) project.

![[homelab-homepage-dashboard-final-layout.png]]
*The completed Homepage dashboard, grouping services by Hardware, VMs, Productivity, and Personal.*
