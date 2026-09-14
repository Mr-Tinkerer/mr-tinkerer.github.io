---
title: Open WebUI Setup
description: Standing up Open WebUI as a chat front-end for llama.cpp, with web search via SearXNG.
tags:
  - Homelab
  - Open-webui
---
[Open WebUI](https://github.com/open-webui/open-webui) runs as a [[Homelab/Logical/Containerization/Tool-choices|Podman]] container on the [[Homelab/Logical/Virtual-machines/Vm-allocation|Local AI VM]], connected to [[Homelab/Platform/Local-AI-VM/Llama.cpp/Serving-as-a-service|llama-server]] as its backend, reachable at `LLMs.uhhhhh` through [[Homelab/Software/Nginx/Reverse-proxy-setup|Nginx]]. The maintained quadlet is kept in the infrastructure repo: [open-webui.container](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Quadlets/open-webui.container).

The quadlet uses the `ghcr.io/open-webui/open-webui:main-slim` image (the slimmer variant, without bundled extras like Whisper/embedded Ollama), runs with `Network=host` (shares the VM's network namespace directly rather than publishing a mapped port), and persists its data at `%h/open-webui:/app/backend/data`. It also declares `After=searxng.service` and `Requires=searxng.service` — Open WebUI is hard-dependent on [[Homelab/Platform/Local-AI-VM/Searxng/Searxng-setup|SearXNG]] being up first, not just loosely integrated with it at the application-settings level (see "Web search via SearXNG" below).

# Connecting to llama-server

I pointed Open WebUI at llama-server's OpenAI-compatible endpoint under `Settings → Connections`, after which models served by llama-server became selectable from Open WebUI's own model picker.

# Web search via SearXNG

I pointed Open WebUI's web-search integration at the home lab's own [[Homelab/Platform/Local-AI-VM/Searxng/Searxng-setup|SearXNG]] instance. I had to make two settings changes to make search behave predictably with small local models:

- **Function calling set to "legacy"** — most of the tested local models don't reliably know how to call Open WebUI's native search *function*, so search is triggered manually (the legacy explicit search toggle) instead of left to the model's own judgment.
- **Follow-up question generation disabled** — an extra LLM call Open WebUI makes by default to suggest follow-up questions, turned off to save inference time/resources on this CPU-only VM.

See [[Homelab/Platform/Local-AI-VM/Searxng/Searxng-setup|SearXNG]] for the search engine itself.
