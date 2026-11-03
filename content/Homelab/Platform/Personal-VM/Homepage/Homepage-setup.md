---
title: Homepage Setup on Personal VM (openSUSE)
description: Standing up a Homepage dashboard on the Personal VM to avoid typing full home lab hostnames.
tags:
  - Homelab
  - Homepage
---

## Overview

Homepage provides a single bookmarkable dashboard linking to every home lab service, addressing browser behavior with the custom, unregistered TLD that treats bare hostnames as search queries instead of URLs unless `https://` is typed explicitly.

## Architecture

Homepage runs as a Podman quadlet-managed container on the Personal VM (openSUSE), reachable at `home.uhhhhh` through Nginx.

### Infrastructure Repository

Maintained quadlet and config volume are stored in the infrastructure repository:

- [homepage.container](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Personal%20VM/Homepage/Quadlet/homepage.container)
- [homepage.network](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Personal%20VM/Homepage/Quadlet/homepage.network)
- [config directory](https://github.com/Mr-Tinkerer/Project-Dump/tree/main/Homelab/Personal%20VM/Homepage/Volume)

## Configuration Issues

Three separate fixes were required to get the container reachable and writable:

### Volume Configuration

Config changes weren't taking effect until the container was given its own dedicated volume rather than relying on defaults baked into the image.

See [[Homelab/Platform/Personal-VM/Opensuse/Opensuse-gotchas|openSUSE: Gotchas]] for the `firewalld` and SELinux fixes, which are openSUSE OS-level defaults rather than Homepage-specific.

## Integrations

Homepage is connected to several other home lab services via their own integrations:

- **Nextcloud** — Via a long-lived API token
- **OpenWrt** — Via login credentials
- **OpenMediaVault** — Via login credentials
- **Tailscale** — Direct integration
- **Proxmox** — Via an API token (widget did not display data correctly at time of writing)

Custom service icons come from the homarr-labs/dashboard-icons project.

See [dashboard screenshot](Images/homelab-homepage-dashboard-final-layout.png) for the completed Homepage dashboard, grouping services by Hardware, VMs, Productivity, and Personal.
