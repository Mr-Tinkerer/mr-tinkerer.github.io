---
title: Nginx Glossary
description: Terms used across the Nginx bucket.
tags:
  - glossary
---
## Reverse proxy

A server that accepts client requests and forwards them to the correct backend server, so clients only ever see one address/port regardless of how many backends exist. See [Reverse proxy (Wikipedia)](https://en.wikipedia.org/wiki/Reverse_proxy).

## Nginx

A free, open-source web server and reverse proxy. I run it on the OpenWrt router as the single thing listening on ports 80/443 for the whole home lab — it's what lets me type a plain hostname like `https://pve.uhhhhh` instead of an IP and port, by matching the hostname and forwarding the request to the right backend behind the scenes (see [[Homelab/Software/Nginx/Reverse-proxy-setup|Reverse proxy setup]]). It's also what terminates HTTPS using the certificates I issue from my private CA (see [[Homelab/Software/Nginx/Ssl-certificates|SSL certificates]]). See [Nginx (Wikipedia)](https://en.wikipedia.org/wiki/Nginx).

## uhttpd

OpenWrt's default lightweight web server, used to serve the [[Homelab/Software/OpenWrt/Openwrt-glossary#LuCI|LuCI]] admin UI. See [uhttpd (OpenWrt docs)](https://openwrt.org/docs/guide-user/services/webserver/http.uhttpd).

## SSHFS

A tool that mounts a remote directory over SSH as if it were a local filesystem, useful for editing remote config files with a local editor. See [SSHFS (Wikipedia)](https://en.wikipedia.org/wiki/SSHFS).

## Self-signed certificate

An SSL/TLS certificate signed by its own private key rather than a recognized certificate authority; not trusted by clients by default, so every browser flags it with a warning until manually approved on that specific site. I considered this route for the home lab's internal `.uhhhhh` domain but decided against it — it doesn't scale past a handful of sites, since every client has to click through (or manually trust) a warning for each individual hostname separately. That's why I set up a [[Homelab/Software/Nginx/Nginx-glossary#Certificate authority (CA)|private CA]] instead: trust it once per device, and every certificate it signs afterward is trusted automatically. See [Self-signed certificate (Wikipedia)](https://en.wikipedia.org/wiki/Self-signed_certificate).

## Certificate authority (CA)

A trusted entity that signs certificates so clients can verify a site's identity. A **private CA** works the same way but is only trusted by devices where its root certificate has been manually installed. See [Certificate authority (Wikipedia)](https://en.wikipedia.org/wiki/Certificate_authority).
