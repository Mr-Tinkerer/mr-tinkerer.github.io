---
title: Arrrrgh VM Overview
description: Overview of the Arrrrgh Windows Server VM, its role in the home lab, and the media management services it hosts.
tags:
  - Homelab
  - Platform
  - Arrrrgh-VM
  - Windows-Server
---

## Overview

This page documents the Arrrrgh Windows Server VM, its role in the home lab, and the media management services it hosts.

---

## Arrrrgh VM Role and Purpose

The Arrrrgh Windows Server VM provides a dedicated Windows-based platform for media management services including the ARR (App Repository & Remote) stack. It runs on Proxmox as a Windows Server VM configured for torrent downloading, metadata management, and media library organization.

---

## Guest OS

The VM runs Windows Server as its guest operating system. See [[Homelab/Platform/Arrrrgh-VM/Windows Server 2019 Installation and Configuration on Arrrrgh VM|Windows Server]] for OS-level details and virtualization configuration.

---

## Hosted Services

### ARR Stack Configuration

Overview of ARR stack service configuration and integration.

- [[Homelab/Platform/Arrrrgh-VM/ARR stack Configuration|ARR Stack Configuration]]

### Prowlarr

Indexer management and metadata coordination.

- [[Homelab/Platform/Arrrrgh-VM/Prowlarr Setup on Arrrrgh Windows Server|Prowlarr Setup]]
- [[Homelab/Platform/Arrrrgh-VM/ARR Stack Shared Glossary|ARR Stack Shared Glossary]]

### FlareSolverr

CAPTCHA bypass and indexer access tool.

- [[Homelab/Platform/Arrrrgh-VM/FlareSolverr Setup|FlareSolverr Setup]]
- [[Homelab/Platform/Arrrrgh-VM/ARR Stack Shared Glossary|ARR Stack Shared Glossary]]

### Deluge

Torrent client for downloads.

- [[Homelab/Platform/Arrrrgh-VM/Deluge Setup|Deluge Setup]]
- [[Homelab/Platform/Arrrrgh-VM/ARR Stack Shared Glossary|ARR Stack Shared Glossary]]

### Sonarr

TV show management and automation.

- [[Homelab/Platform/Arrrrgh-VM/Sonarr Setup on Arrrrgh Windows Server|Sonarr Setup]]
- [[Homelab/Platform/Arrrrgh-VM/ARR Stack Shared Glossary|ARR Stack Shared Glossary]]

### Radarr

Movie management and automation.

- [[Homelab/Platform/Arrrrgh-VM/Radarr Setup on Arrrrgh Windows Server|Radarr Setup]]
- [[Homelab/Platform/Arrrrgh-VM/ARR Stack Shared Glossary|ARR Stack Shared Glossary]]

### Bazarr

Subtitle management service.

- [[Homelab/Platform/Arrrrgh-VM/Bazarr Setup|Bazarr Setup]]
- [[Homelab/Platform/Arrrrgh-VM/ARR Stack Shared Glossary|ARR Stack Shared Glossary]]

### Jellyfin LXC

Media server container with GPU passthrough for hardware-accelerated transcoding.

- [[Homelab/Platform/Arrrrgh-VM/Jellyfin LXC|Jellyfin LXC]]

### NFS Mount

Media libraries are stored on an NFS share mounted at `/mnt/Arrrrgh` for shared access between services.

---

## Service Dependencies

All ARR stack services are configured to work together through API integration:

- Prowlarr coordinates indexer settings across all services
- Services connect to Prowlarr to sync indexer configurations
- Deluge handles actual downloads, with Sonarr/Radarr managing organization
- Bazarr retrieves subtitles based on Sonarr/Radarr library needs