---
title: ConvertX Installation
description: Running ConvertX as a Podman container on the Productivity VM.
tags:
  - Homelab
  - Convertx
---
[ConvertX](https://github.com/C4illin/ConvertX) is a self-hosted file conversion tool, run as a [[Homelab/Logical/Containerization/Tool-choices|Podman]] quadlet-managed container on the [[Homelab/Logical/Virtual-machines/Vm-allocation|Productivity VM]], reachable at `convertx.uhhhhh` through [[Homelab/Software/Nginx/Reverse-proxy-setup|Nginx]] (see [convertx.conf](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Nginx/conf.d/convertx.conf)). The maintained quadlet is kept in the infrastructure repo: [convertx.container](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Productivity%20VM/convertx/convertx.container), [convertx.network](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Productivity%20VM/convertx/convertx.network).

# Configuration

I configured ConvertX with `ACCOUNT_REGISTRATION=false` and `ALLOW_UNAUTHENTICATED=true` — it runs without login, relying on the home lab's network/[[Homelab/Software/Nginx/Ssl-certificates|HTTPS]] boundary rather than its own auth. I set `AUTO_DELETE_EVERY_N_HOURS=1` to clear uploaded/converted files an hour after they're created.