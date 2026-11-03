---
title: Llama.cpp Serving as a Service on Local-AI VM (Fedora)
description: Running llama-server persistently via systemd and switching between downloaded models at runtime.
tags:
  - Homelab
  - Llama.cpp
---

## Overview

This page covers running llama-server persistently via systemd and switching between models at runtime.

## Manual Server Execution

Run the server manually first to verify configuration:

```bash
llama-server --host 0.0.0.0 -t 4 -c 28762 -m ~/models/Gemma-3-4B_gemma-3-4b-it-Q4_K_M.gguf
```

### Firewall Configuration

The server initially wasn't reachable from another machine on `10.10.0.5:8080` — Fedora's `firewalld` was running and blocking the port. Disabling it resolved this.

## Runtime Model Selection

Passing a directory instead of a single file (`--models-dir ~/models`) lets llama-server's built-in web UI pick any downloaded model at request time, including vision-capable ones, instead of hard-coding one model per server start.

## Systemd Service Configuration

A systemd service keeps `llama-server` running persistently and restarts it on failure. The maintained unit file is stored in the infrastructure repository: [llama-server.service](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Service/llama-server.service).

### ExecStart Configuration

The actual `ExecStart` line:

```
ExecStart=llama-server --host 10.10.0.5 --port 9931 --no-webui -t 4 -c 28762 --models-dir /home/ai/models/
```

### Service Details

- **Fixed LAN IP** — Bound to `10.10.0.5` rather than `0.0.0.0`, with `--no-webui` to save resources once Open WebUI became the front-end
- **Non-default port** — Changed from `8080` to `9931` due to default llama.cpp changing in future release
- **Working Directory** — Set to `/opt/llama.cpp/build/bin`, matching where the source tree was relocated during the build
- **Models directory** — `--models-dir` points at `/home/ai/models/`, letting the server's model picker switch between any downloaded model
- **Restart policy** — `Restart=on-failure` with `RestartSec=1` for a fast, tight restart loop rather than the more permissive `Restart=always` used by Podman quadlets
- **User unit** — No `[Service] User=` is set, and the `[Install]` target is `default.target` rather than `multi-user.target` — installed as a systemd user unit
- **Linger configuration** — `loginctl enable-linger` was set for that user so the user's systemd session (and this service) keeps running even with no active login session

See [[Homelab/Software/Nginx/Reverse-proxy-setup|Nginx]] for reverse proxy access at `LLMs.uhhhhh`.

---

See [[Homelab/Platform/Local-AI-VM/Open-webui/Open-webui-setup|Open WebUI]] for the chat front-end connected to this server.
