---
title: Homelab Platform Overview
description: Overview of all Platform VMs in the home lab, including their roles and hosted services.
tags:
  - Homelab
  - Platform
---

## Overview

This page documents all Platform VMs in the home lab, organized by their roles and hosted services.

## Productivity VM

Debian-based platform for document management and file sharing services.

### Role

Primary productivity platform hosting Nextcloud for document management, Stirling-PDF for PDF manipulation, and ConvertX for e-book conversion.

### Guest OS

- [[Homelab/Platform/Productivity-VM/Debian|Debian]]

### Services

- [[Homelab/Platform/Productivity-VM/Nextcloud|Nextcloud]]
- [[Homelab/Platform/Productivity-VM/Stirling-pdf|Stirling-PDF]]
- [[Homelab/Platform/Productivity-VM/Convertx|ConvertX]]

## Local AI VM

Fedora-based platform for local AI inference and related services.

### Role

Dedicated AI inference platform hosting Llama.cpp for local LLM serving, Open-WebUI for chat interfaces, and SearXNG for search.

### Guest OS

- [[Homelab/Platform/Local-AI-VM/Fedora|Fedora]]

### Services

- [[Homelab/Platform/Local-AI-VM/Llama.cpp|Llama.cpp]]
- [[Homelab/Platform/Local-AI-VM/Open-webui|Open-WebUI]]
- [[Homelab/Platform/Local-AI-VM/Searxng|SearXNG]]

## Personal VM

openSUSE-based platform for personal computing and web browsing.

### Role

Personal workspace providing desktop functionality and personal services.

### Guest OS

- [[Homelab/Platform/Personal-VM/Opensuse|openSUSE]]

### Services

- [[Homelab/Platform/Personal-VM/Homepage|Homepage]]

## Platform Architecture

All Platform VMs run as Proxmox LXC containers following the Podman over Docker architecture. Services within each VM communicate via shared Podman networks.

## Resource Allocation

Resource distribution across Platform VMs is documented in the [[Homelab/Logical/Virtual-machines/Vm-allocation|VM Allocation]] roster.

## Future Expansion

The Platform domain is designed to accommodate additional VMs and services as the home lab evolves. Current infrastructure supports adding new Platform VMs without significant reconfiguration.
