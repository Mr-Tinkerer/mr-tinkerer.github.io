---
title: LightRAG Authentication and API Keys
description: LightRAG authentication configuration, API key retirement, and localhost-only access on the Local-AI-VM Fedora platform.
tags:
  - LightRAG
  - homelab
  - fedora
  - rag
  - authentication
---

## Overview

This page documents the LightRAG authentication configuration on the Local-AI-VM Fedora platform.

---

## LightRAG Auth (Deployed .env)

The deployed `.env` enables both API key and JWT accounts. From `env.example` (the reference document, see [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-and-Docling-Self-Hosted-RAG-Stack#Caveats Up Front|LightRAG Stack]]):

| Variable | Purpose | Deployed |
|---|---|---|
| `LIGHTRAG_API_KEY` | X-API-Key header for programmatic access | set |
| `AUTH_ACCOUNTS` | `user:{bcrypt}hash` list; enables WebUI login | `admin:<bcrypt>` |
| `TOKEN_SECRET` | HS256 signing key >=32 bytes | set |
| `JWT_ALGORITHM` | HS256 | set |
| `TOKEN_EXPIRE_HOURS` | JWT lifetime | 48 |
| `GUEST_TOKEN_EXPIRE_HOURS` | Guest-token lifetime | 24 |
| `TOKEN_AUTO_RENEW` | Sliding-window renewal | true |
| `TOKEN_RENEW_THRESHOLD` | Renew when remaining < x × total | 0.5 |
| `LOGIN_MAX_FAILED_ATTEMPTS` | Before lockout | 5 |
| `LOGIN_LOCKOUT_WINDOW_SECONDS` | Lockout window | 300 |
| `WHITELIST_PATHS` | Endpoints exempt from auth | /health |

### Security Model

- `LIGHTRAG_API_KEY` requires `X-API-Key: <key>` on all API requests.
- `AUTH_ACCOUNTS` + `TOKEN_SECRET` protects `/webui` and `/workspace` (browser login).
- If neither is set, the server has no auth and every endpoint is public. This is only safe on `HOST=127.0.0.1`.
- `WHITELIST_PATHS=/health` keeps `/health` public for liveness probes.
- The default `WHITELIST_PATHS` (when unset) is `/health` if `LIGHTRAG_API_KEY` is set, or `/health,/api/*` if it isn't (Ollama-client compatibility).
- Ollama-compatible `/api/*` routes follow `LIGHTRAG_API_KEY`, not `AUTH_ACCOUNTS` — Ollama clients can't log in. Set the key before exposing.

The deployed `HOST` value is not in the `.env` file (defaults to `0.0.0.0`). The container publishes port 9621 on the host — so the host is network-accessible. The auth setup is the protection.

---

## llama-server API Keys (Retired)

**Status:** API keys were **used and working** for llama.cpp, but were **retired due to annoyance** from managing them.

**Current Access:** The API now **restricts access to localhost only** as a middle ground solution.

The `.env` previously sent a bearer token (`<redacted>`) on every llama-server request. The uploaded `llama-server.service` (available at [[https://github.com/Mr-Tinkerer/Project-Dump/tree/main/Archives/RAG|GitHub RAG Archive]]) did not pass `--api-key` to llama-server, which means:

- Either the server accepted any key (no `--api-key` set), in which case the token was ignored.
- Or the live unit had `--api-key` uncommented, and the deployed service file was a stale draft.

The commented-out alternatives in the uploaded file showed `--api-key "<redacted>"` was tried. The correct configuration was settled during the RAG stack's active deployment.

**Note:** The RAG stack files, including the original `llama-server.service`, are available at [[https://github.com/Mr-Tinkerer/Project-Dump/tree/main/Archives/RAG|GitHub RAG Archive]].

---

## Podman Deployment

For Podman Quadlet deployment details that can be reused across all Podman environments (homelab & laptop), see [[Generic-Info/Podman/Podman-Quadlet|Podman Quadlet]].

For Podman Compose deployment information specific to the LightRAG stack (historical reference), see [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-Podman-Compose|Podman Compose]].

---

## Known Discrepancies

### LightRAG-* Role Presets in presets.ini

The uploaded `presets.ini` has the `[LightRAG-Logic]`, `[LightRAG-Vision]`, `[LightRAG-Embedding]` sections commented out. The deployed `.env` uses those exact role names as the model strings.

If the uploaded `presets.ini` is the live one, requests for `LightRAG-Logic` will 404 (`model not found`). If the live `presets.ini` has them uncommented, everything is fine.

### Containerfile :latest Tag

The uploaded `Containerfile` says `FROM ghcr.io/docling-project/docling-serve-cpu:latest`. The user reports the live image is digest-pinned. The live `Containerfile` should be updated to `FROM ghcr.io/docling-project/docling-serve-cpu@sha256:<digest>`.
