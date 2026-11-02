---
title: ARR Stack Configuration
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

See [[Prowlarr Setup]] for detailed configuration.

### FlareSolverr

See [[FlareSolverr Setup]] for detailed configuration.

### Deluge

See [[Deluge Setup]] for detailed configuration.

### Sonarr

See [[Sonarr Setup]] for detailed configuration.

### Radarr

See [[Radarr Setup]] for detailed configuration.

### Bazarr

See [[Bazarr Setup]] for detailed configuration.

---

## Integration

Services integrate through API connections managed by Prowlarr:

1. **Prowlarr** coordinates indexer settings
2. **FlareSolverr** provides CAPTCHA bypass for restricted indexers
3. **Deluge** handles actual downloads
4. **Sonarr/Radarr** manage download organization
5. **Bazarr** retrieves subtitles based on library needs