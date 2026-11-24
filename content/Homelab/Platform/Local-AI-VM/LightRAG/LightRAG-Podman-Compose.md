---
title: LightRAG Podman Compose Deployment
description: Historical Podman Compose deployment for LightRAG, retained for reference and comparison with Quadlet.
tags:
  - LightRAG
  - Podman
  - homelab
  - fedora
  - rag
  - compose
---

## Overview

This page documents the Podman Compose deployment for the LightRAG stack. This approach has been **retired in favor of Quadlet**, but is retained here for historical reference and comparison.

**Current Status:** Quadlet is now the recommended deployment method. See [[Generic-Info/Podman/Podman-Quadlet|Podman Quadlet]] for the current deployment guide.

---

## Compose Stack

The compose stack was used during the early deployment phases (Era 0 through G). It provided easy temporary configuration but was replaced by Quadlet for permanent deployment.

### File Layout

```
~/.config/containers/systemd/
├── docling.build          # Build unit
├── docling.container      # Docling service
├── lightrag.container     # LightRAG service
└── rag.network            # Bridge network
```

---

## Compose Deployment

### Activation

```bash
mkdir -p ~/.config/containers/systemd
# copy unit files there
podman compose down         # if any old containers are running
systemctl --user daemon-reload
systemctl --user start docling.service lightrag.service
loginctl enable-linger ai   # boot-start without login
```

### Podman Compose vs. Quadlet

| Feature | Podman Compose | Quadlet |
|---|---|---|
| Persistence | Temporary, requires manual restart | Permanent, systemd-managed |
| Configuration | Compose YAML | systemd unit files |
| Boot-start | Requires `enable` | Automatic with `loginctl enable-linger` |
| Best for | Testing, temporary setups | Production, long-running services |

**Why Quadlet was chosen:** Quadlet units are permanent, integrate with systemd, and provide better boot-start persistence without requiring manual intervention.

---

## RAG Stack Files

All files for the RAG stack are available at [[https://github.com/Mr-Tinkerer/Project-Dump/tree/main/Archives/RAG|GitHub RAG Archive]].

This includes:
- `compose.yml` - Original compose configuration
- `lightrag.container` - LightRAG Quadlet unit
- `docling.container` - Docling Quadlet unit
- `docling.build` - Docling build unit
- `rag.network` - Bridge network definition
- `llama-server.service` - llama.cpp router service
- `.env` - Environment configuration
- `Containerfile` - Docling patch source

---

## Related Documentation

- [[Homelab/Platform/Local-AI-VM/LightRAG/Auth-and-API-Keys|Auth and API Keys]] - Authentication configuration
- [[Generic-Info/Podman/Podman-Quadlet|Podman Quadlet]] - Current deployment method
- [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-and-Docling-Self-Hosted-RAG-Stack|LightRAG Stack]] - Comprehensive guide