---
title: Nginx Reverse Proxy Setup
description: Running Nginx on the OpenWrt router to give each home lab service a clean hostname instead of a raw IP:port.
tags:
  - Homelab
  - Nginx
  - Networking
---

## Overview

This page covers running Nginx on the OpenWrt router to give each home lab service a clean hostname instead of a raw IP:port.

## Motivation

OpenWrt's DNS server can resolve hostnames to IPs, but can't route by port — so without a proxy, reaching Proxmox meant typing `https://pve.uhhhhh:8006` instead of just `https://pve.uhhhhh`. A reverse proxy fixes this by taking the plain hostname and forwarding the request to the right backend IP/port behind the scenes.

## Port 80/443 Configuration

OpenWrt's own web UI (uhttpd) listens on 80/443 by default, which conflicts with Nginx wanting the same ports. Since clients need to reach Nginx on the standard ports, uhttpd was moved instead:

- Changed listen addresses in `/etc/config/uhttpd` to `0.0.0.0:888` (HTTP) and `0.0.0.0:444` (HTTPS)
- Restarted uhttpd: `/etc/init.d/uhttpd restart`

### Nginx Installation

Nginx was installed via `apk add nginx` and started/enabled with:

```bash
/etc/init.d/nginx enable && /etc/init.d/nginx start
```

## Disabling UCI Auto-Generation

Freshly installed, Nginx on OpenWrt returns `403 Forbidden` by default — OpenWrt ships a custom preset and dynamically regenerates Nginx's config directory from UCI settings on every start.

### UCI Approach

Adding a site the "OpenWrt way" means adding UCI keys such as:

```bash
uci set nginx.srv_grafana=server
uci set nginx.srv_grafana.uci_enable='true'
uci set nginx.srv_grafana.server_name='grafana.raspberrypi.home'
# ... additional configuration
uci commit nginx
/etc/init.d/nginx restart
```

*(Illustrative example values from the guide adapted — not this home lab's actual domain names.)*

This UCI-driven approach was dropped in favor of plain Nginx config files. Running `uci set nginx.global.uci_enable=false && uci commit nginx` disables the auto-generation feature, after which Nginx behaves like a standard install with config living entirely under `/etc/nginx`.

## Remote Configuration Editing

VS Code's remote-SSH extension didn't work against OpenWrt (missing tooling it depends on), so SSHFS was used instead:

- After installing the `openssh-sftp-server` package on OpenWrt
- `sshfs OpenWrt:/etc/nginx nginx` mounts the remote config directory locally for editing with any normal editor

### Git Integration

The `/etc/nginx` directory was turned into a git repository for change tracking. Since it's mounted and edited as a different user than the one that created the mount, Git initially refused to trust the directory — resolved with `git config --global --add safe.directory nginx`.

### Maintained Configuration

The maintained Nginx site configuration is stored in a separate repository rather than pasted inline: [Nginx config](https://github.com/Mr-Tinkerer/openwrt-nginx).

## Services Added Since Initial Setup

As new VM-hosted services came online, each got its own `conf.d` site file following the same pattern (HTTP→HTTPS redirect, then a `proxy_pass` to the service's VM IP:port):

- nextcloud.conf
- office.nextcloud.conf (Collabora)
- stirling.conf
- convertx.conf
- home.conf (Homepage)
- LLMs.conf (llama.cpp/Open WebUI)
- search.conf (SearXNG)

See the SSL certificates page for enabling HTTPS on these proxied sites.
