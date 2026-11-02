---
title: Open-WebUI on Local-AI-VM (Read-Only Instance)
description: Read-only copy of Open-WebUI that syncs chat history from the primary Daily Driver Laptop instance.
tags:
  - Homelab
  - Local-AI-VM
  - Open-WebUI
---

## Overview

This entry documents the read-only Open-WebUI instance on the Local-AI VM. This instance serves as a backup copy for accessing chat logs when the primary Daily Driver Laptop instance is unavailable. It does not have its own independent setup; instead, it relies on database synchronization from the laptop.

---

## Primary Instance

The primary Open-WebUI instance runs on the Daily Driver Laptop (Gigabyte Gaming A16). This instance acts as the remote chat interface for the laptop's local LLM conversations.

---

## Database Synchronization

The Local-AI VM instance receives chat history through automatic database synchronization from the laptop.

### Synchronization Scripts

The following files handle database synchronization from laptop to Local-AI VM:

- [Open-WebUI-DB-Sync.sh](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Daily%20Driver%20Laptop/Local%20AI/Open-WebUI%20DB%20Sync/Open-WebUI-DB-Sync.sh) — Synchronization script
- [Open_WebUI_DB_Sync.service](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Daily%20Driver%20Laptop/Local%20AI/Open-WebUI%20DB%20Sync/Open_WebUI_DB_Sync.service) — Systemd service definition
- [Open_WebUI_DB_Sync.timer](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Daily%20Driver%20Laptop/Local%20AI/Open-WebUI%20DB%20Sync/Open_WebUI_DB_Sync.timer) — Systemd timer for scheduled execution

### Service Activation

Enable and start the synchronization timer on the Daily Driver Laptop:

```bash
systemctl --user enable --now Open-WebUI-DB-Sync.timer
```

### Synchronization Flow

1. Chat messages created on the laptop are stored in the SQLite database
2. The sync timer runs periodically on the Local-AI VM
3. The sync script copies chat history from the laptop to the Local-AI VM
4. The Local-AI VM instance displays synced chat logs for read-only access

---

### Issue: Model Search Failure

**Observed Issue:** Open-WebUI version 0.11.4-slim removed embedding support required for web search functionality, causing the model to fail when performing online searches.

**Solution:** The `main` branch version of Open-WebUI was used to restore embedding support and web search capabilities.

---

## Open-WebUI Preferences

Disable automatic follow-up generation in the interface settings to prevent the model from generating follow-up questions after each response.