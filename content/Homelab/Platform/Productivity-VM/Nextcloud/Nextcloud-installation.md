---
title: Nextcloud Installation
description: Running Nextcloud as a set of Podman quadlets on the Productivity VM.
tags:
  - Homelab
  - Nextcloud
---
# Why Podman, and why quadlets

Nextcloud runs in containers on the [[Homelab/Logical/Virtual-machines/Vm-allocation|Productivity VM]] (Debian) via [[Homelab/Logical/Containerization/Containerization-glossary#Podman|Podman]] rather than Docker, using the official [Nextcloud Docker image](https://github.com/nextcloud/docker) as a base with local modifications. See [[Homelab/Logical/Containerization/Tool-choices|Tool-choice philosophy]] for why I chose Podman/Quadlet for every containerized service in this domain, not just Nextcloud.

# Stack layout

The stack is four containers on one Podman network (`nextcloud-net`):

- **`nextcloud`** — the app itself, built from a locally customized image (`nextcloud-custom`).
- **`mariadb`** — the database backend.
- **`redis`** — caching/locking backend.
- **`collabora`** — Nextcloud Office, a separate container (see [[Homelab/Platform/Productivity-VM/Nextcloud/Collabora-office|Collabora Office]]).

The maintained quadlet files are kept in the infrastructure repo rather than pasted here: [nextcloud.container](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Productivity%20VM/nextcloud/quadlets/nextcloud.container), [mariadb.container](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Productivity%20VM/nextcloud/quadlets/mariadb.container), [redis.container](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Productivity%20VM/nextcloud/quadlets/redis.container), [collabora.container](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Productivity%20VM/nextcloud/quadlets/collabora.container), [nextcloud.network](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Productivity%20VM/nextcloud/quadlets/nextcloud.network).

# Domain, DNS, and HTTPS

Nextcloud is reachable at `nextcloud.uhhhhh`, with a DNS record under [[Homelab/Logical/Home-network/Dns-naming-scheme|the home lab's custom DNS]] and HTTPS via [[Homelab/Software/Nginx/Ssl-certificates|Nginx's private CA]]. The domain and expected trusted proxies had to be explicitly configured in Nextcloud's own settings (`trusted_domains`, `trusted_proxies`) so it would accept requests forwarded by Nginx and correctly detect HTTPS — see [[Homelab/Platform/Productivity-VM/Nextcloud/Nextcloud-gotchas|Gotchas]] for the specific errors this produced before it was fixed.

# Storage

The [[Homelab/Software/OpenMediaVault/Shares|Documents share]] from the NAS is mounted on the Productivity VM host and bind-mounted into the `nextcloud` container, so files placed there are exposed through Nextcloud without duplicating storage.

See [[Homelab/Platform/Productivity-VM/Nextcloud/Collabora-office|Collabora Office]] for the document-editing add-on, and [[Homelab/Platform/Productivity-VM/Nextcloud/Nextcloud-gotchas|Gotchas]] for the certificate-trust and redirect issues hit along the way.
