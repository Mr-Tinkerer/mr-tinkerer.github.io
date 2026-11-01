---
title: SearXNG Setup
description: Running a private SearXNG instance for local LLM web search without third-party API dependency.
tags:
  - Homelab
  - Searxng
---

## Overview

SearXNG provides a private, self-hosted search backend for local LLMs, eliminating dependency on third-party search APIs.

## Architecture

SearXNG runs as a Podman container on the Local AI VM.

- **Access:** `search.uhhhhh` through Nginx
- **Configuration:** [search.conf](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Nginx/conf.d/search.conf)
- **Quadlet:** [searxng.container](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Quadlets/searxng.container)

## Container Configuration

### Network Publishing

Unlike Open WebUI's `Network=host` setup, SearXNG uses port mapping:

- `PublishPort=8888:8080` — host port 8888 maps to container's internal port 8080

### Volume Persistence

Two separate volumes persist:

- `%h/searxng/config/:/etc/searxng` — SearXNG configuration (e.g., `settings.yml`)
- `%h/searxng/data/:/var/cache/searxng` — Cache data

### Service Dependencies

Open WebUI's quadlet declares `Requires=`/`After=` dependency on SearXNG, ensuring SearXNG starts before Open WebUI.

See Open WebUI for integration with the chat front-end.
