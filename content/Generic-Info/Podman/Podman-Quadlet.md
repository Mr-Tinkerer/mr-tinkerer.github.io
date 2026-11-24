---
title: Podman Quadlet Deployment
description: Reusable Podman Quadlet deployment guide for homelab and laptop environments. See [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-Podman-Compose|Podman Compose]] for historical reference.
tags:
  - Podman
  - systemd
  - homelab
  - laptop
  - quadlet
---

## Overview

This page documents Podman Quadlet deployment, a permanent and production-ready way to run Podman containers as systemd user services. Quadlet units are placed in `~/.config/containers/systemd/` and automatically generate the corresponding systemd units.

**Why Quadlet:** Quadlet provides a more permanent and maintainable solution compared to `podman compose`. It integrates with systemd, supports boot-start persistence, and is ideal for long-running services on both homelab VMs and laptop systems.

---

## File Layout

For a multi-container stack, organize your units like this:

```
~/.config/containers/systemd/
├── <service1>.build          # Build unit (for custom images)
├── <service1>.container      # Service 1 container
├── <service2>.container      # Service 2 container
└── <network>.network         # Bridge network (optional)
```

**Example from LightRAG:**
```
~/.config/containers/systemd/
├── docling.build          # Build unit (for custom images)
├── docling.container      # Docling service
├── lightrag.container     # LightRAG service
└── rag.network            # Bridge network
```

---

## Activation

```bash
mkdir -p ~/.config/containers/systemd
# copy unit files there
podman compose down         # if any old containers are running
systemctl --user daemon-reload
systemctl --user start <service1>.service <service2>.service
loginctl enable-linger <username>   # boot-start without login
```

**Example from LightRAG:**
```bash
systemctl --user start docling.service lightrag.service
loginctl enable-linger ai   # boot-start without login
```

---

## Quadlet Quirks and Caveats

1. **Quadlet-generated units are transient** — `systemctl --user enable` does not work on them. Use `[Install] WantedBy=default.target` (which Quadlet honours) plus `loginctl enable-linger <username>`.

2. **`depends_on` → `Requires=` + `After=`.** Start ordering only, not readiness. Container A calling Container B at startup tolerates this.

3. **`mem_limit: 3g` → `Memory=3g`.** Rootless caveat: if cgroup memory delegation is not enabled, the limit is silently ignored. Check:
   ```bash
   cat /sys/fs/cgroup/user.slice/user-$(id -u).slice/user@$(id -u).service/cgroup.controllers
   ```
   Look for `memory`.

4. **Quadlet does not resolve `./`.** Use absolute paths (e.g., `/home/ai/rag/...`).

5. **Build unit:** `<service>.build.service` builds before `<service>.service` starts. Rebuild after an image update:
   ```bash
   systemctl --user restart <service>.build.service
   systemctl --user restart <service>.service
   ```

6. **Container name consistency:** If your compose stack used `container_name: <name>`, the Quadlet unit should explicitly set `ContainerName=<name>` to preserve the DNS name on the bridge network.

---

## Optional <stack>.target

A `.target` unit (in `~/.config/systemd/user/`) groups the containers for start/stop-as-one:

```ini
[Unit]
Description=<Stack Name> (<service1> + <service2>)
Wants=<service1>.service <service2>.service

[Install]
WantedBy=default.target
```

In each container unit, add `PartOf=<stack>.target` under `[Unit]` and `WantedBy=<stack>.target` under `[Install]`. Then `systemctl --user start <stack>.target`. Not strictly needed for two services (Service A's `Requires=` already pulls Service B in), but useful if the stack grows.

**Example from LightRAG:**
```ini
[Unit]
Description=RAG stack (Docling + LightRAG)
Wants=docling.service lightrag.service

[Install]
WantedBy=default.target
```

---

## Logs

```bash
# Interleaved journal for both containers
journalctl --user -u <service1>.service -u <service2>.service -f

# Warnings and above only
journalctl --user -u <service1> -u <service2> -p warning -f

# Since last boot
journalctl --user -u <service1> -u <service2> -b
```

Optional shared syslog tag: add `LogDriver=journald` and `PodmanArgs=--log-opt tag=<stack>` under `[Container]` in each unit, then `journalctl --user -t <stack> -f`.

`journalctl -u <stack>.target` shows the target's own messages, not member output. Use the shell function pattern or the tag.

---

## Related Documentation

- [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-Podman-Compose|Podman Compose]] - Historical compose deployment
- [[Homelab/Platform/Local-AI-VM/LightRAG/Auth-and-API-Keys|Auth and API Keys]] - Authentication configuration
- [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-and-Docling-Self-Hosted-RAG-Stack|LightRAG Stack]] - Comprehensive guide