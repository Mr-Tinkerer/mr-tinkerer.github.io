---
title: Llama.cpp on Local-AI-VM Overview
description: Overview of llama.cpp installation, benchmarks, configuration, and LightRAG integration on the Local-AI-VM Fedora platform.
tags:
  - llama.cpp
  - homelab
  - fedora
  - local-ai-vm
  - overview
---

## Overview

This page documents llama.cpp usage on the Local-AI-VM Fedora platform, covering installation, benchmarks, configuration, and integration with LightRAG for self-hosted RAG applications.

---

## Installation and Configuration

See [[Build and CPU Tuning|Build and CPU Tuning]] for details on building llama.cpp from source with native CPU optimizations.

---

## Benchmarks

See [[Benchmarks|Benchmarks]] for CPU-only inference performance benchmarks across various models and context sizes.

---

## Serving as a Service

See [[Serving as a Service|Serving as a Service]] for documentation on running llama.cpp as a service for other applications.

---

## LLM Services Configuration and API Setup

See [[LLM Services Configuration and API Setup on Local-AI VM (Fedora)|LLM Services Configuration]] for details on configuring llama.cpp to serve as an OpenAI-compatible API.

---

## LightRAG Integration

See [[Homelab/Platform/Local-AI-VM/LightRAG|LightRAG Bucket]] for an overview of the LightRAG and Docling self-hosted RAG stack.

### LightRAG and Docling RAG Stack

See [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-and-Docling-Self-Hosted-RAG-Stack|LightRAG and Docling RAG Stack]] for comprehensive documentation covering:

- Era timeline and deployment history
- Hardware and repository layout
- Deployed stack configuration (Quadlet, containers, presets)
- Docling parsing (90% full-bleed filter fix)
- LightRAG parser routing and the `!` option
- JSON extraction and hallucination prevention
- Entity deduplication pipeline
- Decision-model benchmark for entity merge
- Auth, API keys, and Quadlet deployment
- Known discrepancies and open items

---

## Glossary

See [[Llama-cpp-glossary|Glossary]] for terms specific to running llama.cpp on the CPU-only Local-AI-VM.

### LightRAG-Specific Glossary

See [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-Glossary|LightRAG Glossary]] for terms specific to the LightRAG and Docling RAG stack.
