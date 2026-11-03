---
title: ARR Stack Configuration on Arrrrgh Windows Server
description: Overview of ARR stack service configuration and integration on the Arrrrgh VM.
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

This entry documents the configuration and integration of the ARR (App Repository & Remote) stack services on the Arrrrgh Windows Server VM.

---

## Folder Structure

Recommended folder structure for data organization:

```
data
├── torrents
│   ├── books
│   ├── movies
│   ├── music
│   ├── tv
│   ├── ISOs
│   └── software
├── usenet
│   ├── incomplete
│   └── complete
│       ├── books
│       ├── movies
│       ├── music
│       └── tv
└── media
    ├── books
    ├── movies
    ├── music
    ├── tv
    ├── ISOs
    └── software
```

PowerShell command to recreate this structure:

```powershell
$data = "Z:\The Seven Seas"

$folders = @(
    "$data\torrents\books",
    "$data\torrents\movies",
    "$data\torrents\music",
    "$data\torrents\tv",
    "$data\torrents\ISOs",
    "$data\torrents\software",
    "$data\usenet\incomplete",
    "$data\usenet\complete\books",
    "$data\usenet\complete\movies",
    "$data\usenet\complete\music",
    "$data\usenet\complete\tv",
    "$data\media\books",
    "$data\media\movies",
    "$data\media\music",
    "$data\media\tv",
    "$data\media\ISOs",
    "$data\media\software"
)

$folders | ForEach-Object {
    New-Item -ItemType Directory -Path $_ -Force | Out-Null
}
```

---

## Service Configuration

### Prowlarr

See [[Homelab/Platform/Arrrrgh-VM/Prowlarr Setup|Prowlarr]] for detailed configuration.

### FlareSolverr

See [[Homelab/Platform/Arrrrgh-VM/FlareSolverr Setup|FlareSolverr]] for detailed configuration.

### Deluge

See [[Homelab/Platform/Arrrrgh-VM/Deluge Setup|Deluge]] for detailed configuration.

### Sonarr

See [[Homelab/Platform/Arrrrgh-VM/Sonarr Setup|Sonarr]] for detailed configuration.

### Radarr

See [[Homelab/Platform/Arrrrgh-VM/Radarr Setup|Radarr]] for detailed configuration.

### Bazarr

See [[Homelab/Platform/Arrrrgh-VM/Bazarr Installation and Configuration on Arrrrgh VM|Bazarr Installation and Configuration]] for detailed configuration.

---

## Integration

Services integrate through API connections managed by Prowlarr:

1. **Prowlarr** coordinates indexer settings
2. **FlareSolverr** provides CAPTCHA bypass for restricted indexers
3. **Deluge** handles actual downloads
4. **Sonarr/Radarr** manage download organization
5. **Bazarr** retrieves subtitles based on library needs