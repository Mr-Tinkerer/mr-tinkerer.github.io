---
title: LightRAG and Docling Self-Hosted RAG Stack — Comprehensive Guide
description: Complete comprehensive guide to the LightRAG and Docling self-hosted RAG stack on the Local-AI-VM Fedora platform.
tags:
  - LightRAG
  - Docling
  - homelab
  - fedora
  - rag
---

This is a comprehensive guide that consolidates all LightRAG and Docling documentation. For focused reading, see the individual pages:

- [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-Core-Server|LightRAG Core Server]] - Core server configuration and deployment
- [[Homelab/Platform/Local-AI-VM/LightRAG/Docling-Parsing|Docling Parsing]] - Document parsing and the 90% full-bleed filter fix
- [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-Parser-Routing|LightRAG Parser Routing]] - Parser routing and the `!` option
- [[Homelab/Platform/Local-AI-VM/LightRAG/JSON-Extraction-and-Hallucination|JSON Extraction]] - JSON extraction and hallucination prevention
- [[Homelab/Platform/Local-AI-VM/LightRAG/Entity-Deduplication|Entity Deduplication]] - Entity deduplication pipeline
- [[Homelab/Platform/Local-AI-VM/LightRAG/Decision-Model-Benchmark|Decision-Model Benchmark]] - Entity merge verification benchmark
- [[Homelab/Platform/Local-AI-VM/LightRAG/Auth-and-API-Keys|Auth and API Keys]] - Authentication and Podman deployment
- [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-Podman-Compose|Podman Compose]] - Historical compose deployment

For a quick overview, see [[Homelab/Platform/Local-AI-VM/LightRAG/Index|LightRAG Bucket Overview]].

---

## Overview

This page documents the design, debugging, and deployment of a self-hosted RAG pipeline on an 8 GB CPU-only Fedora VM (Dell OptiPlex 5060 host), using LightRAG as the RAG server, docling-serve as the document parser, and three local llama.cpp model roles served through a single router. Rootless Podman is deployed as Quadlet units. Auth is enabled with API key and JWT accounts.

The RAG stack was later archived after the realization that direct .md file access better served the original goal of a quiz agent over an Obsidian notes vault. The deployed architecture remains in place but is not the active focus.

---

## Document Scope, Conventions, and Era Timeline

This document consolidates a multi-chat series documenting the design, debugging, and eventual archiving of a self-hosted LightRAG + Docling RAG stack on an 8 GB CPU-only Fedora VM.

### Era Timeline

| Era | Key Characteristics | Primary Source |
|---|---|---|
| 0 | Initial deployment. Gemma-3-4B LLM; Qwen3-Embedding-8B (3072d); host.containers.internal; planned rag-sync.py; no ! image rules | File 10 |
| A | wrap_images.py introduced; stock docling-serve-cpu; DOCLING_SERVE_ENABLE_UI=true; MiniCPM-V-2 VLM | File 6 |
| B | ! image rules added; Qwen2-VL-7B VLM; wrap_images.py still in use | File 7 |
| C | Docling patched (90%-filter disabled); wrap_images.py retired; VLM -> Qwen2.5-VL-3B; Tesseract OCR; force_ocr investigation | File 1 (Docling) |
| D | Laptop-GPU offload experiment (shelved). Root cause of unreachability was nftables, not LightRAG. Led to the split-server recommendation | File 8 |
| E | JSON extraction; Qwen3.5-4B thinkless on 9933; split servers 9931 / 9933 / 9932; MAX_EXTRACT_INPUT_TOKENS=8000; dedup design spec | Files 3, 4 |
| H | Dedupe script (dedupe_entities.py) evolved to complete form; layers 1-4 | File 1 (dedupe) |
| G | Quadlet conversion | File 9 |
| Deployed | Single router on 9931 with presets.ini role aliases; real API keys; auth block; Quadlet; pinned Containerfile; Docling port not exposed | .env, Containerfile, dedupe_entities.py, docling.build, docling.container, lightrag.container, rag.network, presets.ini, llama-server.service |
| F | RAG archived; quiz agent pivots to direct .md file access | File 5 |

### Caveats Up Front

1. The deployed .env and llama-server.service have a small inconsistency (llama-server binds 127.0.0.1 only; .env addresses host.containers.internal:9931). See [[Homelab/Platform/Local-AI-VM/LightRAG/Auth-and-API-Keys#Known Discrepancies|Auth and API Keys]].
2. The uploaded presets.ini has the LightRAG-* role sections commented out, while the deployed .env uses those role names. The live presets.ini presumably has them uncommented. See [[Homelab/Platform/Local-AI-VM/LightRAG/Auth-and-API-Keys#LightRAG-* Role Presets in presets.ini|Auth and API Keys]].
3. The uploaded Containerfile still says :latest; the user reports the live image is digest-pinned. See [[Homelab/Platform/Local-AI-VM/LightRAG/Auth-and-API-Keys#Containerfile :latest Tag|Auth and API Keys]].

---

## Focused Documentation

For detailed information on specific topics, see the individual pages:

### Server and Deployment

- [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-Core-Server|LightRAG Core Server]] - Core server configuration, authentication, and deployment details

### Parsing

- [[Homelab/Platform/Local-AI-VM/LightRAG/Docling-Parsing|Docling Parsing]] - Document parsing, the 90% full-bleed filter fix, and OCR settings

### Routing

- [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-Parser-Routing|LightRAG Parser Routing]] - Parser routing configuration and the `!` option for skipping extraction

### Extraction

- [[Homelab/Platform/Local-AI-VM/LightRAG/JSON-Extraction-and-Hallucination|JSON Extraction]] - JSON extraction mode and hallucination prevention

### Deduplication

- [[Homelab/Platform/Local-AI-VM/LightRAG/Entity-Deduplication|Entity Deduplication]] - Entity deduplication pipeline with dedupe_entities.py

### Benchmarking

- [[Homelab/Platform/Local-AI-VM/LightRAG/Decision-Model-Benchmark|Decision-Model Benchmark]] - Entity merge verification benchmark

### Authentication

- [[Homelab/Platform/Local-AI-VM/LightRAG/Auth-and-API-Keys|Auth and API Keys]] - Authentication configuration and Quadlet deployment

---

## Related Documentation

- [[Homelab/Platform/Local-AI-VM/LightRAG|LightRAG Bucket]] - LightRAG bucket overview
- [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-Glossary|LightRAG Glossary]] - LightRAG-specific glossary
