---
title: LightRAG Core Server
description: Core LightRAG server configuration, authentication, and RAG retrieval modes on the Local-AI-VM Fedora platform.
tags:
  - LightRAG
  - homelab
  - fedora
  - rag
---

## Overview

This page documents the core LightRAG server configuration on the Local-AI-VM Fedora platform. LightRAG is a self-hosted RAG (Retrieval-Augmented Generation) server that combines knowledge graph extraction with vector embedding for improved retrieval quality.

---

## Server Configuration

The LightRAG server runs as a Podman container on the Local-AI-VM, exposed on port 9621. It uses a bridge network (`rag.network`) for inter-container communication with Docling.

### Environment Variables

Key environment variables configured in `.env`:

| Variable | Purpose | Deployed Value |
|---|---|---|
| `LIGHTRAG_API_KEY` | X-API-Key header for programmatic access | Set |
| `AUTH_ACCOUNTS` | `user:{bcrypt}hash` list; enables WebUI login | `admin:<bcrypt>` |
| `TOKEN_SECRET` | HS256 signing key >=32 bytes | Set |
| `JWT_ALGORITHM` | HS256 | Set |
| `TOKEN_EXPIRE_HOURS` | JWT lifetime | 48 |
| `GUEST_TOKEN_EXPIRE_HOURS` | Guest-token lifetime | 24 |
| `TOKEN_AUTO_RENEW` | Sliding-window renewal | true |
| `TOKEN_RENEW_THRESHOLD` | Renew when remaining < x × total | 0.5 |
| `LOGIN_MAX_FAILED_ATTEMPTS` | Before lockout | 5 |
| `LOGIN_LOCKOUT_WINDOW_SECONDS` | Lockout window | 300 |
| `WHITELIST_PATHS` | Endpoints exempt from auth | /health |

### Security Model

- `LIGHTRAG_API_KEY` requires `X-API-Key: <key>` on all API requests
- `AUTH_ACCOUNTS` + `TOKEN_SECRET` protects `/webui` and `/workspace` (browser login)
- If neither is set, the server has no auth and every endpoint is public
- `WHITELIST_PATHS=/health` keeps `/health` public for liveness probes
- The default `WHITELIST_PATHS` (when unset) is `/health` if `LIGHTRAG_API_KEY` is set, or `/health,/api/*` if it isn't (Ollama-client compatibility)
- Ollama-compatible `/api/*` routes follow `LIGHTRAG_API_KEY`, not `AUTH_ACCOUNTS` — Ollama clients can't log in. Set the key before exposing.

---

## RAG Retrieval Modes

LightRAG supports multiple retrieval modes:

- **naive:** Uses only vector chunks, no graph traversal
- **local:** Uses knowledge graph for retrieval
- **global:** Uses global knowledge graph for retrieval
- **hybrid:** Combines graph and vector retrieval
- **mix:** Uses both graph and vector retrieval (WebUI default)

The WebUI uses `mix` by default, combining both graph and vector retrieval for optimal results.

---

## Container Deployment

LightRAG runs as a rootless Podman container deployed via Quadlet. The container configuration is in `lightrag.container`:

```ini
[Unit]
Description=LightRAG server
Requires=docling.service
After=docling.service

[Container]
ContainerName=lightrag
Image=ghcr.io/hkuds/lightrag:latest
Network=rag.network
PublishPort=9621:9621
EnvironmentFile=/home/ai/rag/.env
Volume=/home/ai/rag/lightRAG/rag_storage:/app/data/rag_storage:Z
Volume=/home/ai/rag/lightRAG/inputs:/app/data/inputs:Z
Volume=/home/ai/rag/.env:/app/.env:Z

[Service]
Restart=always

[Install]
WantedBy=default.target
```

### Quadlet Deployment

**Why Quadlet:** The user switched from `podman compose` (temporary, easy tweaks) to Quadlet units (permanent). Compose was retired.

**File layout:**

```
~/.config/containers/systemd/
├── docling.build          # Build unit
├── docling.container      # Docling service
├── lightrag.container     # LightRAG service
└── rag.network            # Bridge network
```

**Activation:**

```bash
mkdir -p ~/.config/containers/systemd
# copy unit files there
podman compose down         # if any old containers are running
systemctl --user daemon-reload
systemctl --user start docling.service lightrag.service
loginctl enable-linger ai   # boot-start without login
```

**Quadlet quirks and caveats:**

1. **Quadlet-generated units are transient** — `systemctl --user enable` does not work on them. Use `[Install] WantedBy=default.target` (which Quadlet honours) plus `loginctl enable-linger ai`.
2. **`depends_on` → `Requires=` + `After=`.** Start ordering only, not readiness. LightRAG calling Docling at parse time tolerates this.
3. **`mem_limit: 3g` → `Memory=3g`.** Rootless caveat: if cgroup memory delegation is not enabled, the limit is silently ignored. Check:
   ```bash
   cat /sys/fs/cgroup/user.slice/user-$(id -u).slice/user@$(id -u).service/cgroup.controllers
   ```
   Look for `memory`.
4. **Quadlet does not resolve `./`.** Use absolute paths (`/home/ai/rag/...`).
5. **Build unit:** `docling-build.service` builds before `docling.service` starts. Rebuild after a docling-serve update:
   ```bash
   systemctl --user restart docling-build.service
   systemctl --user restart docling.service
   ```
6. **Container name change:** the compose stack used `container_name: docling`. Quadlet default would be `docling-serve.service` or similar; the deployed `docling.container` explicitly sets `ContainerName=docling`, preserving the `DOCLING_ENDPOINT=http://docling:5001` DNS name on `rag.network`.

**Logs:**

```bash
# Interleaved journal for both containers
journalctl --user -u docling.service -u lightrag.service -f

# Warnings and above only
journalctl --user -u docling -u lightrag -p warning -f

# Since last boot
journalctl --user -u docling -u lightrag -b
```

Optional shared syslog tag: add `LogDriver=journald` and `PodmanArgs=--log-opt tag=rag` under `[Container]` in each unit, then `journalctl --user -t rag -f`.

`journalctl -u rag.target` shows the target's own messages, not member output. Use the shell function pattern or the tag.

---

## Archived Status

The RAG stack was archived (Era F) after the realization that the original goal — a quiz agent over an Obsidian notes vault — was better served by direct `.md` file access than by the full RAG pipeline. The deployed architecture remains in place but is not the active focus. The Quizzer preset in `presets.ini` (Qwen3.5-2B, ctx 32768) is the post-RAG quiz agent's model.
