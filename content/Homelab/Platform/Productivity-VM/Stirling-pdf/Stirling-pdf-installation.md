---
title: Stirling PDF Installation
description: Running Stirling PDF as a Podman container on the Productivity VM.
tags:
  - Homelab
  - Stirling-pdf
---
[Stirling PDF](https://www.stirlingpdf.com/) runs as a single [[Homelab/Logical/Containerization/Tool-choices|Podman]] quadlet-managed container on the [[Homelab/Logical/Virtual-machines/Vm-allocation|Productivity VM]], reachable at `pdfs.uhhhhh` through [[Homelab/Software/Nginx/Reverse-proxy-setup|Nginx]] (see [stirling.conf](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Nginx/conf.d/stirling.conf)). The maintained quadlet is kept in the infrastructure repo: [stirling.container](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Productivity%20VM/stirling/stirling.container), [stirling.network](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Productivity%20VM/stirling/stirling.network).