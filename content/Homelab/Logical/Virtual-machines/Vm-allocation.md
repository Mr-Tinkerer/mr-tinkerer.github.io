---
title: VM Allocation on Proxmox Hypervisor
description: The roster of VMs running on the Proxmox hypervisor, what each one is for, and which OS it runs.
tags:
  - Homelab
  - Networking
---

## Overview

This page documents the roster of VMs running on the Proxmox hypervisor, their roles, and their guest OS choices.

## Why This is Logical, Not Part of Proxmox

Proxmox is the software used to host these VMs, but the scheme of which VM does what — one per functional role, each with its own OS chosen for that role — is a convention that spans Proxmox, every guest OS, and every service in this domain. It has its own bucket for the same reason [[Homelab/Logical/Home-network/Addressing-plan|Home-network]] has its own bucket: it's a shared plan other buckets participate in, not a single installed thing.

## The Roster

| VM | Role | Guest OS | Notes |
|----|------|----------|-------|
| Monitoring VM | Monitoring the rest of the home lab's services | Not yet decided | Only the guest OS has been installed so far; no monitoring software is configured yet |
| Productivity VM | Nextcloud, Stirling PDF, ConvertX | Debian | Runs Podman for all its containers |
| Personal VM | Homepage dashboard and other personal-use services | openSUSE | Ships with `firewalld` and SELinux both enabled by default — see the openSUSE Gotchas page for the gotchas this caused |
| Local AI VM | Local LLM inference (llama.cpp) and its web front-ends (Open WebUI, SearXNG) | Fedora | CPU set to `host` passthrough mode so the guest sees the physical CPU's real instruction set (AVX2), instead of a generic profile that might not include it — see the Fedora CPU configuration page |
| Arrrrgh VM | Media/download-stack services (planned; nothing configured yet) | Windows Server 2022 | Sized smaller (4GB RAM / 32GB storage) than the shared VM baseline. Only the guest OS has been installed so far |

## Access Configuration

Each VM is kept reachable at a fixed address on the home lab's LAN subnet and proxied through Nginx under its own hostname per the DNS naming scheme.

See [[Homelab/Platform/Personal-VM/Homepage/Homepage-setup|Homepage]] for a single-dashboard view of this whole roster.
