---
title: LLMs Other Tweaks
description: Additional configuration and setup notes for LLM services including llama-server and LightRAG.
tags:
  - Homelab
  - LLM
  - llama-server
  - LightRAG
  - API
---

## Overview

This entry documents additional tweaks and configurations for LLM services, including unified llama-server services for multiple systems, API key management, and authentication setup for LightRAG.

---

## Unified llama-Server Services

### Service Consolidation

Combined all `llama-server` services into one configuration for both the Dell Optiplex and Daily Driver Laptop systems. This centralizes management and ensures consistent configuration across environments.

- [Dell Optiplex llama-server.service](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Service/llama-server.service)
- [Daily Driver Laptop llama-server.service](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Daily%20Driver%20Laptop/Local%20AI/Services/llama-server.service)

### Prefix Configuration

Setup prefixes for system identification:

- [Dell Optiplex presets.ini](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Service/presets.ini)
- [Daily Driver Laptop presets.ini](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Daily%20Driver%20Laptop/Local%20AI/presets.ini)

These presets allow the services to distinguish between different deployment environments.

---

## API Key Management

### llama.cpp API Keys

Set API keys for llama.cpp on both systems:

- [Dell Optiplex llama-server.service](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Service/llama-server.service)
- [Daily Driver Laptop llama-server.service](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Daily%20Driver%20Laptop/Local%20AI/Services/llama-server.service)

API keys provide authentication for external services accessing the llama.cpp API.

### LightRAG API Keys

Set API keys for LightRAG service and reference the llama.cpp API keys in the `.env` file:

- [LightRAG .env](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/RAG/.env)

This allows LightRAG to authenticate with llama.cpp for model inference.

---

## LightRAG Authentication

### Account Setup

Setup authentication accounts for LightRAG service in the `.env` configuration file:

- [LightRAG .env](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/RAG/.env)

Authentication ensures only authorized users can access LightRAG functionality.
```
