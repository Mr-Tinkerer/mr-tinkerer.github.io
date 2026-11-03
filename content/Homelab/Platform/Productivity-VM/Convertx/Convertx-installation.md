---
title: ConvertX Installation on Productivity VM (Debian)
description: Running ConvertX as a Podman container on the Productivity VM for file conversion.
tags:
  - Homelab
  - Convertx
---

## Overview

ConvertX provides self-hosted file conversion capabilities, running as a Podman quadlet-managed container on the Productivity VM.

- **Access:** `convertx.uhhhhh` through Nginx
- **Configuration:** [convertx.conf](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Nginx/conf.d/convertx.conf)
- **Quadlet:** [convertx.container](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Productivity%20VM/convertx/convertx.container), [convertx.network](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Productivity%20VM/convertx/convertx.network)

## Configuration

### Authentication Settings

- `ACCOUNT_REGISTRATION=false` — Disabled user registration
- `ALLOW_UNAUTHENTICATED=true` — Allows unauthenticated access

ConvertX relies on the home lab's network/HTTPS boundary rather than its own authentication.

### File Cleanup

`AUTO_DELETE_EVERY_N_HOURS=1` — Uploaded and converted files are deleted one hour after creation.
