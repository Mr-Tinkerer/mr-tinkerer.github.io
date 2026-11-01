---
title: Local-AI-VM Overview
description: Overview of the Local-AI VM, its role in the home lab, and the AI services it hosts.
tags:
  - Homelab
  - Platform
  - Local-AI-VM
---

## Overview

This page documents the Local-AI VM, its role in the home lab, and the AI services it hosts.

## Local-AI VM Role and Purpose

The Local-AI VM provides a dedicated Fedora-based platform for local AI inference and related services. It runs on Proxmox as a LXC container optimized for CPU-based machine learning workloads without GPU passthrough.

## Guest OS

The VM runs Fedora as its guest operating system, configured with CPU pinning and virtual CPU type optimization for inference workloads. See [[Homelab/Platform/Local-AI-VM/Fedora|Fedora]] for OS-level details, including CPU configuration and system tuning.

## Hosted Services

### Llama.cpp

Local LLM inference engine serving various large language models with CPU-optimized compilation for maximum performance without GPU acceleration.

- [[Homelab/Platform/Local-AI-VM/Llama.cpp|Llama.cpp]]

### Open-WebUI

Web-based interface for interacting with local LLMs, providing chat interfaces and model management.

- [[Homelab/Platform/Local-AI-VM/Open-webui|Open-WebUI]]

### SearXNG

Privacy-focused metasearch engine that aggregates results from multiple search providers.

- [[Homelab/Platform/Local-AI-VM/Searxng|SearXNG]]

## Service Dependencies

All services run as Podman containers on Fedora, following the Podman over Docker architecture. Services communicate via a shared Podman network.

Llama.cpp serves as the primary inference engine, with Open-WebUI providing the user interface and SearXNG offering search capabilities. The inference workload determines resource allocation priorities.

## Performance Considerations

The VM is configured for CPU-only inference, with llama.cpp compiled with native CPU instruction set optimization (AVX2, FMA). See [[Homelab/Platform/Local-AI-VM/Llama.cpp/Build-and-cpu-tuning|Llama.cpp Build and CPU tuning]] for build details and performance verification.

Resource allocation is tuned for inference workloads, prioritizing CPU threads over GPU capabilities. The [[Homelab/Logical/Virtual-machines/Vm-allocation|VM allocation roster]] documents resource distribution across all VMs.

## Future Expansion

The VM is designed to accommodate additional AI-related services as the local AI stack evolves. Current capacity supports adding 1-2 additional lightweight inference or tooling services without significant performance impact.
