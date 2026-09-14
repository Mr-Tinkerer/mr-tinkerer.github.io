---
title: Serving as a service
description: Running llama-server persistently via systemd, and switching between downloaded models at runtime.
tags:
  - Homelab
  - Llama.cpp
---
# Running the server manually first

```bash
llama-server --host 0.0.0.0 -t 4 -c 28762 -m ~/models/Gemma-3-4B_gemma-3-4b-it-Q4_K_M.gguf
```

The server initially wasn't reachable from another machine on `10.10.0.5:8080` — Fedora's `firewalld` (see [[Homelab/Platform/Personal-VM/Homepage/Homepage-glossary#firewalld|firewalld]]) was running and blocking the port; disabling it resolved this.

# Picking a model at runtime

Passing a directory instead of a single file (`--models-dir ~/models`) lets llama-server's own built-in web UI pick any downloaded model at request time, including vision-capable ones, instead of hard-coding one model per server start.

# Running it as a systemd service

A systemd service keeps `llama-server` running persistently and restarts it on failure. The maintained unit file is kept in the infrastructure repo: [llama-server.service](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Service/llama-server.service).

The actual `ExecStart` line:

```
ExecStart=llama-server --host 10.10.0.5 --port 9931 --no-webui -t 4 -c 28762 --models-dir /home/ai/models/
```

Notable details baked into that service:

- **Bound to a fixed LAN IP** (`10.10.0.5`) rather than `0.0.0.0`, and `--no-webui` is set to save a small amount of resource once [[Homelab/Platform/Local-AI-VM/Open-webui/Open-webui-setup|Open WebUI]] became the actual front-end.
- **Non-default port (`9931`).** Changed from the default `8080` due to [default llama.cpp changing in future release](https://github.com/ggml-org/llama.cpp/pull/26508).
- **`WorkingDirectory=/opt/llama.cpp/build/bin`**, matching where the source tree was relocated during the build (see [[Homelab/Platform/Local-AI-VM/Llama.cpp/Build-and-cpu-tuning#Installing to the path|Build and CPU tuning]]) — `ExecStart` invokes the bare `llama-server` command name rather than a full path, relying on the `/usr/local/bin` symlink being on `PATH`.
- **`--models-dir` points at `/home/ai/models/`**, letting the server's own model picker switch between any downloaded model (see "Picking a model at runtime" above) rather than hard-coding one file.
- **`Restart=on-failure` with `RestartSec=1`** — a fast, tight restart loop rather than the more permissive `Restart=always` used by the Podman quadlets below.
- **No `[Service] User=`** is set, and the unit's `[Install]` target is `default.target` rather than `multi-user.target` — consistent with this being installed as a systemd **user** unit (run under a specific login user's own systemd instance) rather than a system-wide unit. **`loginctl enable-linger`** was set for that user so the user's systemd session (and this service) keeps running even with no active login session — this detail lives outside the unit file itself (it's a `loginctl` setting on the host), so it can't be re-verified from the unit file alone, but it's consistent with the `default.target`/no-`User=` shape of the file.

Reachable through [[Homelab/Software/Nginx/Reverse-proxy-setup|Nginx]] at `LLMs.uhhhhh` (see [LLMs.conf](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Nginx/conf.d/LLMs.conf)).

See [[Homelab/Platform/Local-AI-VM/Open-webui/Open-webui-setup|Open WebUI]] for the chat front-end connected to this server.
