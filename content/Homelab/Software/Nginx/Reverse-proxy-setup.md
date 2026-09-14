---
title: Reverse proxy setup
description: Running Nginx on the OpenWrt router to give each home lab service a clean hostname instead of a raw IP:port.
tags:
  - Homelab
  - Nginx
  - Networking
---
# Why

[[Homelab/Software/OpenWrt/Dhcp-and-dns-configuration|OpenWrt's DNS server]] can resolve hostnames to IPs, but can't route by port — so without a proxy, reaching Proxmox meant typing `https://pve.uhhhhh:8006` instead of just `https://pve.uhhhhh`. I fixed that with a [[Homelab/Software/Nginx/Nginx-glossary#Reverse proxy|reverse proxy]], which takes the plain hostname and forwards the request to the right backend IP/port behind the scenes.

# Freeing up ports 80/443

OpenWrt's own web UI ([[Homelab/Software/Nginx/Nginx-glossary#uhttpd|uhttpd]]) listens on 80/443 by default, which conflicts with Nginx wanting the same ports. Since clients need to reach Nginx on the standard ports, I moved uhttpd instead — I changed its listen addresses in `/etc/config/uhttpd` to `0.0.0.0:888` (HTTP) and `0.0.0.0:444` (HTTPS), then ran `/etc/init.d/uhttpd restart`.

I then installed Nginx itself via `apk add nginx` and started/enabled it with `/etc/init.d/nginx enable && /etc/init.d/nginx start`.

# OpenWrt's Nginx defaults get in the way

Freshly installed, Nginx on OpenWrt returns `403 Forbidden` by default — OpenWrt ships a custom preset, and (unlike a typical Linux distro) dynamically regenerates Nginx's config directory from [[Homelab/Software/OpenWrt/Openwrt-glossary#UCI|UCI]] settings on every start. Adding a site the "OpenWrt way" means adding UCI keys such as:

```bash
uci set nginx.srv_grafana=server
uci set nginx.srv_grafana.uci_enable='true'
uci set nginx.srv_grafana.server_name='grafana.raspberrypi.home'
uci set nginx.srv_grafana.include='conf.d/grafana.locations'
uci set nginx.srv_grafana.ssl_certificate='/etc/nginx/ssl/wildcard.raspberrypi.home.crt'
uci set nginx.srv_grafana.ssl_certificate_key='/etc/nginx/ssl/wildcard.raspberrypi.home.key'
uci add_list nginx.srv_grafana.listen='443 ssl'
uci add_list nginx.srv_grafana.listen='[::]:443 ssl'
uci set nginx.srv_grafana.ssl_session_cache='shared:SSL:32k'
uci set nginx.srv_grafana.ssl_session_timeout='64m'
uci commit nginx
/etc/init.d/nginx restart
```

*(Illustrative example values from the guide I adapted this from — not this home lab's actual domain names.)*

I dropped this UCI-driven approach in favor of plain Nginx config files: running `uci set nginx.global.uci_enable=false && uci commit nginx` disables the auto-generation feature, after which Nginx behaves like a standard install with config living entirely under `/etc/nginx`.

# Editing config remotely

VS Code's remote-SSH extension didn't work against OpenWrt (missing tooling it depends on), so I used [[Homelab/Software/Nginx/Nginx-glossary#SSHFS|SSHFS]] instead: after installing the `openssh-sftp-server` package on OpenWrt, `sshfs OpenWrt:/etc/nginx nginx` mounts the remote config directory locally for editing with any normal editor.

I turned the `/etc/nginx` directory into a git repository for change tracking. Since it's mounted and edited as a different user than the one that created the mount, Git initially refused to trust the directory — I resolved that with `git config --global --add safe.directory nginx`.

I keep the maintained Nginx site configuration itself in a separate repository rather than pasting it here: [Nginx config](https://github.com/Mr-Tinkerer/openwrt-nginx).

---
# Services added since the initial setup

As new VM-hosted services came online, I gave each its own `conf.d` site file following the same pattern (HTTP→HTTPS redirect, then a `proxy_pass` to the service's VM IP:port): [nextcloud.conf](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Nginx/conf.d/nextcloud.conf), [office.nextcloud.conf](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Nginx/conf.d/office.nextcloud.conf) (Collabora), [stirling.conf](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Nginx/conf.d/stirling.conf), [convertx.conf](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Nginx/conf.d/convertx.conf), [home.conf](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Nginx/conf.d/home.conf) (Homepage), [LLMs.conf](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Nginx/conf.d/LLMs.conf) (llama.cpp/Open WebUI), and [search.conf](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Nginx/conf.d/search.conf) (SearXNG).

See [[Homelab/Software/Nginx/Ssl-certificates|SSL certificates]] for enabling HTTPS on these proxied sites.
