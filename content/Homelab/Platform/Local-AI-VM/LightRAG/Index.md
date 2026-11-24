---
title: LightRAG and Docling RAG Stack Overview
description: Overview of the LightRAG and Docling self-hosted RAG stack on the Local-AI-VM Fedora platform.
tags:
  - LightRAG
  - Docling
  - homelab
  - fedora
  - rag
  - overview
---

## Overview

This page documents the LightRAG and Docling self-hosted RAG stack deployed on the Local-AI-VM Fedora platform. The stack uses LightRAG as the RAG server, docling-serve as the document parser, and three local llama.cpp model roles served through a single router.

The RAG stack was later archived after the realization that direct .md file access better served the original goal of a quiz agent over an Obsidian notes vault. The deployed architecture remains in place but is not the active focus.

---

## Documentation

See [[LightRAG and Docling Self-Hosted RAG Stack|LightRAG and Docling RAG Stack]] for comprehensive documentation covering:

- Era timeline and deployment history
- Hardware and repository layout
- Deployed stack configuration (Quadlet, containers, presets)
- Docling parsing (90% full-bleed filter fix)
- LightRAG parser routing and the `!` option
- JSON extraction and hallucination prevention
- Entity deduplication pipeline
- Decision-model benchmark for entity merge
- Auth, API keys, and Podman deployment
- Known discrepancies and open items

For Podman Compose deployment information (historical reference), see [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-Podman-Compose|Podman Compose]].

---

## Related Documentation

- [[Homelab/Platform/Local-AI-VM/Llama.cpp|Llama.cpp on Local-AI-VM Overview]] - Main Llama.cpp documentation
- [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-Glossary|LightRAG Glossary]] - LightRAG-specific terms
