---
title: Nextcloud Installation
description: Running Nextcloud as a set of Podman quadlets on the Productivity VM for document management and file sharing.
tags:
  - Homelab
  - Nextcloud
---

## Overview

This guide covers Nextcloud installation and configuration on the Productivity VM, including container setup, network configuration, and storage integration.

## Architecture

Nextcloud runs as Podman containers on the Productivity VM (Debian), following the Podman over Docker architecture decision.

The official Nextcloud Docker image serves as the base with local modifications. See the Tool-choice philosophy page for the rationale behind Podman/Quadlet across all containerized services.

### Container Stack

The stack consists of four containers on a single Podman network (`nextcloud-net`):

- **`nextcloud`** — The application itself, built from a locally customized image (`nextcloud-custom`)
- **`mariadb`** — Database backend
- **`redis`** — Caching and locking backend
- **`collabora`** — Nextcloud Office, a separate container

### Quadlet Configuration

Maintained quadlet files are stored in the infrastructure repository rather than pasted inline:

- nextcloud.container
- mariadb.container
- redis.container
- collabora.container
- nextcloud.network

## Network Configuration

### Access

Nextcloud is reachable at `nextcloud.uhhhhh` with a DNS record under the home lab's custom DNS.

### HTTPS

HTTPS is provided via Nginx's private CA.

### Trusted Proxies

The domain and expected trusted proxies must be explicitly configured in Nextcloud settings (`trusted_domains`, `trusted_proxies`) to accept requests forwarded by Nginx and correctly detect HTTPS. See the Gotchas page for specific errors before resolution.

## Storage

### NAS Integration

The Documents share from the NAS is mounted on the Productivity VM host and bind-mounted into the `nextcloud` container. Files placed in the NAS share are exposed through Nextcloud without duplicating storage.

## Additional Resources

- [[Homelab/Platform/Productivity-VM/Nextcloud/Collabora-office|Collabora Office]] — Document editing add-on
- [[Homelab/Platform/Productivity-VM/Nextcloud/Nextcloud-gotchas|Gotchas]] — Certificate trust and redirect issues
