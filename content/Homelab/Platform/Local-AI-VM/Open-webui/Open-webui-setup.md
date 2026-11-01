---
title: Open WebUI Setup
description: Standing up Open WebUI as a chat front-end for llama.cpp with web search via SearXNG.
tags:
  - Homelab
  - Open-webui
---

## Overview

Open WebUI runs as a Podman container on the Local AI VM, connected to llama-server as its backend. Accessible at `LLMs.uhhhhh` through Nginx.

## Container Configuration

The quadlet uses the `ghcr.io/open-webui/open-webui:main-slim` image (the slimmer variant without bundled extras like Whisper/embedded Ollama).

### Network Configuration

Runs with `Network=host` — shares the VM's network namespace directly rather than publishing a mapped port.

### Data Persistence

Persists data at `%h/open-webui:/app/backend/data`.

### Service Dependencies

Declares `After=searxng.service` and `Requires=searxng.service` — Open WebUI is hard-dependent on SearXNG being up first, not just loosely integrated at the application-settings level.

See [[Homelab/Platform/Local-AI-VM/Searxng/Searxng-setup|SearXNG]] for the search engine itself.

## llama-server Integration

Open WebUI is pointed at llama-server's OpenAI-compatible endpoint under `Settings → Connections`, after which models served by llama-server become selectable from Open WebUI's model picker.

## Web Search via SearXNG

Open WebUI's web-search integration is pointed at the home lab's SearXNG instance. Two settings changes were required for predictable search behavior with small local models:

### Function Calling

Function calling set to "legacy" — most tested local models don't reliably know how to call Open WebUI's native search function, so search is triggered manually (the legacy explicit search toggle) instead of left to the model's judgment.

### Follow-up Question Generation

Follow-up question generation disabled — an extra LLM call Open WebUI makes by default to suggest follow-up questions, turned off to save inference time/resources on this CPU-only VM.
