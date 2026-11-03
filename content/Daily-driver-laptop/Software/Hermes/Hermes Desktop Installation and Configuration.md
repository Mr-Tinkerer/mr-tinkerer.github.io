---
title: Hermes Agent Desktop Installation on CachyOS
description: Installation and configuration of Hermes Agent desktop on Linux with custom LLM endpoint integration.
tags:
  - Homelab
  - Hermes
  - AI
  - LLM
  - Qwen
---

## Overview

This guide details the installation and configuration of Hermes Agent desktop on Linux, including custom LLM endpoint setup and task execution.

---

## Installation

Install Hermes Desktop on the laptop:

```bash
sudo pacman -S hermes-agent-desktop
```

---

## Initial Configuration

### Quick Tour

Run the quick tour to familiarize yourself with the interface.

### Custom LLM Endpoint Setup

Navigate to Settings → Providers → Custom Endpoints and add an endpoint for local LLM access.

### Model Configuration

Configure the model to create an introduction file:

1. Specify the model: `qwen3.5 4b`
2. Request creation of introduction file at `/home/pengmania/.hermes/introduction.md`
3. Create a separate introduction specifically for the Qwen3.5 model (not the agent)

**Note:** When requesting file creation, verify the output location. The model may write to `/home/pengmania/downloads/` instead of `/home/pengmania/Downloads/`.

### Context Size Configuration

Create a prefix to give Qwen3.5-9B a context size of `501760` tokens (approximately 20 GB of RAM).

---

## Use Case

The configured Hermes instance runs on the Daily Driver Laptop (desktop version). It was used to improve the Quartz wiki documentation.

**Note:** No custom configuration files are used. This setup belongs to the laptop domain, not the Homelab platform.

---

## Troubleshooting

### File Location Mismatch

When requesting file creation, the model may write to different directories than specified. Verify the actual output location and move files if necessary.

### Context Size Requirements

Large context sizes (e.g., 501760 tokens for Qwen3.5-9B) require significant RAM allocation (~20 GB). Ensure sufficient system resources are available.
