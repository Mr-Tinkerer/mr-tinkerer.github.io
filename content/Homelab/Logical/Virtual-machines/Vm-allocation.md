---
title: VM allocation
description: The roster of VMs running on the Proxmox hypervisor, what each one is for, and which OS it runs.
tags:
  - Homelab
  - Networking
---
# Why this is Logical, not part of Proxmox

[[Homelab/Software/Proxmox/Proxmox-installation|Proxmox]] is the software I use to host these VMs, but the *scheme* of which VM does what — one per functional role, each with its own OS chosen for that role — is a convention that spans Proxmox, every guest OS, and every service in this domain. I gave it its own bucket for the same reason I did for [[Homelab/Logical/Home-network/Addressing-plan|Home-network]]: it's a shared plan other buckets participate in, not a single installed thing.

# The roster

| VM | Role | Guest OS | Notes |
|---|---|---|---|
| Monitoring VM | Monitoring the rest of the home lab's services | Not yet decided | Only the guest OS has been installed so far; no monitoring software is configured yet. |
| Productivity VM | [[Homelab/Platform/Productivity-VM/Nextcloud/Nextcloud-installation|Nextcloud]], [[Homelab/Platform/Productivity-VM/Stirling-pdf/Stirling-pdf-installation|Stirling PDF]], [[Homelab/Platform/Productivity-VM/Convertx/Convertx-installation|ConvertX]] | Debian | Runs Podman for all its containers. |
| Personal VM | [[Homelab/Platform/Personal-VM/Homepage/Homepage-setup|Homepage]] dashboard and other personal-use services | openSUSE | Ships with `firewalld` and SELinux both enabled by default — see [[Homelab/Platform/Personal-VM/Opensuse/Opensuse-gotchas|openSUSE: Gotchas]] for the gotchas this caused. |
| Local AI VM | Local LLM inference ([[Homelab/Platform/Local-AI-VM/Llama.cpp/Build-and-cpu-tuning|llama.cpp]]) and its web front-ends ([[Homelab/Platform/Local-AI-VM/Open-webui/Open-webui-setup|Open WebUI]], [[Homelab/Platform/Local-AI-VM/Searxng/Searxng-setup|SearXNG]]) | Fedora | CPU set to `host` passthrough mode so the guest sees the physical CPU's real instruction set (AVX2), instead of a generic profile that might not include it — see [[Homelab/Platform/Local-AI-VM/Fedora/Cpu-configuration|Fedora: CPU configuration]]. |
| Arrrrgh VM | Media/download-stack services (planned; nothing configured yet) | Windows Server 2022 | Sized smaller (4GB RAM / 32GB storage) than the shared VM baseline. Only the guest OS has been installed so far. |

I keep each VM reachable at a fixed address on [[Homelab/Logical/Home-network/Addressing-plan|the home lab's LAN subnet]] and proxied through [[Homelab/Software/Nginx/Reverse-proxy-setup|Nginx]] under its own hostname per [[Homelab/Logical/Home-network/Dns-naming-scheme|the DNS naming scheme]]. See [[Homelab/Platform/Personal-VM/Homepage/Homepage-setup|Homepage]] for a single-dashboard view of this whole roster.
