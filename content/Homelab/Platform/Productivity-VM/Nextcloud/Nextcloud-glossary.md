---
title: Nextcloud Glossary
description: Terms used across the Nextcloud bucket.
tags:
  - glossary
---
## Nextcloud

A self-hosted file sync, sharing, and collaboration platform — essentially a self-hosted alternative to Google Drive/Dropbox, with desktop/mobile sync clients, file sharing links, and an app ecosystem (calendar, contacts, and, on this VM, in-browser office document editing via [[Homelab/Platform/Productivity-VM/Nextcloud/Nextcloud-glossary#Collabora Online|Collabora Online]]). It's the centerpiece of the Productivity VM: I use it to keep files synced across my own devices and to expose the NAS's Documents share through a normal file-sync client instead of raw network shares. See [Nextcloud (official site)](https://nextcloud.com/).

## Collabora Online

An open-source online office suite used by Nextcloud Office to provide in-browser document editing. See [Collabora Online (official site)](https://www.collaboraoffice.com/collabora-online/).

## Trusted proxies

A Nextcloud setting listing which reverse-proxy IPs/subnets are allowed to set forwarding headers (`X-Forwarded-For`, `X-Forwarded-Proto`); without it, Nextcloud can't safely tell a legitimate proxied request from a spoofed one. See [Reverse proxy configuration (Nextcloud docs)](https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/reverse_proxy_configuration.html).
