---
title: ARR Stack Services Overview on Arrrrgh Windows Server
description: Overview of the ARR (App Repository & Remote) stack services for media management on the Arrrrgh VM.
tags:
  - Homelab
  - Arrrrgh-VM
  - Prowlarr
  - FlareSolverr
  - Deluge
  - Sonarr
  - Radarr
  - Bazarr
---

## Overview

The ARR (App Repository & Remote) stack is a collection of interconnected media management services running on the Arrrrgh Windows Server VM. The stack automates torrent downloading, metadata management, and media library organization.

---

## Services

### ARR Stack Services Overview

Overview of ARR stack service configuration and integration.

- [[Homelab/Platform/Arrrrgh-VM/ARR stack Configuration on Arrrrgh Windows Server|ARR stack Configuration]]

### Prowlarr

Indexer management and metadata coordination.

- [[Homelab/Platform/Arrrrgh-VM/Prowlarr Setup|Prowlarr Setup]]

### FlareSolverr

CAPTCHA bypass and indexer access tool.

- [[Homelab/Platform/Arrrrgh-VM/FlareSolverr Setup|FlareSolverr Setup]]

### Deluge

Torrent client for downloads.

- [[Homelab/Platform/Arrrrgh-VM/Deluge Setup|Deluge Setup]]

### Sonarr

TV show management and automation.

- [[Homelab/Platform/Arrrrgh-VM/Sonarr Setup|Sonarr Setup]]

### Radarr

Movie management and automation.

- [[Homelab/Platform/Arrrrgh-VM/Radarr Setup|Radarr Setup]]

### Bazarr

Subtitle management service.

- [[Homelab/Platform/Arrrrgh-VM/Bazarr Installation and Configuration on Arrrrgh VM|Bazarr Installation and Configuration]]

---

## Service Dependencies

All ARR stack services integrate through API connections:

1. **Prowlarr** coordinates indexer settings across all services
2. **Services** connect to Prowlarr to sync indexer configurations
3. **Deluge** handles actual downloads
4. **Sonarr/Radarr** manage download organization and metadata
5. **Bazarr** retrieves subtitles based on Sonarr/Radarr library needs