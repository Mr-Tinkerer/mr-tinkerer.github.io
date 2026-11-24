---
title: JSON Extraction and Hallucination Prevention
description: JSON extraction mode configuration, Qwen3.5-4B thinkless model selection, and hallucination prevention in LightRAG.
tags:
  - LightRAG
  - homelab
  - fedora
  - rag
  - extraction
---

## Overview

This page documents the JSON extraction configuration and hallucination prevention in LightRAG on the Local-AI-VM Fedora platform.

---

## The Delimiter → JSON Cascade

Gemma-3-4B with the default delimiter-based extraction format on the firewall/iptables chunk:

- Extracted 36 entities.
- Every relation was rejected. Result: 36 orphan nodes with zero edges.
- Repeated format failures: `Complete delimiter can not be found`, `found 21/4 fields on ENTITY`, `found 3/5 fields on RELATION`.
- Occasionally looped: repeated `iptables -A OUTPUT ...` ~150 times until `finish_reason=length`.

The cause: the delimiter format is not robust enough for a small model on hierarchical/definitional content. The model also copied format placeholders from the extraction prompt (`<source_entity>`, `<target_entity>`) into entity names when the chunk was near-empty.

**The fix:** `ENTITY_EXTRACTION_USE_JSON=true`. JSON is structured and the parser can validate it. The same firewall/iptables chunk went from 0 relations to 34. Final graph quality (era E, 12 GB run): ~109 entities, ~102 relations.

### Token Budget Interaction

JSON output is more verbose. With `OPENAI_LLM_MAX_TOKENS=3072`, chunk 001 truncated. Raising to 5120 fixed it. Both `OPENAI_LLM_MAX_TOKENS` and `OPENAI_LLM_MAX_COMPLETION_TOKENS` are set to 5120 — different LightRAG code paths read different names, so setting both is defensive. The startup log shows both honored.

---

## Model Choice

| Model | Peak RAM | Verdict |
|---|---|---|
| Gemma-3-4B | ~2.5 GB | Baseline. Weak; format leaks; hallucinates on empty chunks. |
| Llama-3.1-8B | ~10 GB with VLM | Cleaner graph but OOMs on 8 GB when paired with a 7B VLM. |
| **Qwen3.5-4B (thinkless)** | ~2.5 GB | **Selected.** Reasoning mode adds nothing to structured extraction and would be very slow on CPU. |

`--reasoning off --reasoning-budget 0` is critical. Without it, Qwen3.5-4B spends its context budget on chain-of-thought before emitting any output. In the LLM verifier benchmark, thinking mode produced `content: ""` on every response — the reasoning block consumed all tokens, then the response truncated at `finish_reason: length`.

---

## Invented Entities from Near-Empty Chunks

When an image was ingested without `!` (era A/B), the extraction LLM saw a chunk containing only `<drawing .../>` (roughly 88 tokens of markup) and invented entities:

- Tux → SpaceX, Musk, Tesla, Falcon 9, Mars, Starlink.
- Bullet image → Apple Inc., Cupertino, Macbeth, William Shakespeare.
- ISOs screenshot → hallucinated distro names.

Tracing every stray entity in `kv_store_entity_chunks.json` to `chunk-000` confirmed the extraction chunk was the source, not `mm-drawing-000` (the VLM description chunk). The VLM was innocent.

### Mechanism

A 4B model given a near-empty chunk copies few-shot examples from the extraction prompt. You can see the leak: `<source_entity>Elon Musk` in the entity name field.

### Fixes (in order of impact)

1. `!` on image rules — suppresses extraction entirely for image chunks.
2. `ENTITY_EXTRACTION_USE_JSON=true` — reduces format-leak failures.
3. Giving the chunk real text — the `wrap_images.py` wrapped Markdown had a `# Tux with Gun.png` heading that the extractor could work from.

---

## Thin Graph from Images

If extraction does run on an image chunk (e.g., PDF-embedded images), the resulting entities are literal object names ("penguin", "gun", "holding"). Descriptive attributes ("green hill", "clouds", "gun pointed at camera") live in the VLM description chunk, retrievable by vector search but not by graph traversal. This is correct behaviour, not a bug — but it means graph-only queries miss image content entirely.
