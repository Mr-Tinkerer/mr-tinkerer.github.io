---
title: Entity Deduplication with dedupe_entities.py
description: Entity deduplication pipeline using dedupe_entities.py, including rule-based normalization, embedding similarity, and LLM verification.
tags:
  - LightRAG
  - homelab
  - fedora
  - rag
  - deduplication
---

## Overview

This page documents the entity deduplication pipeline using `dedupe_entities.py` on the Local-AI-VM Fedora platform. LightRAG's built-in entity auto-merge (`ENABLE_ENTITY_AUTO_MERGE=true`) is non-functional in the tested version. The post-ingestion `dedupe_entities.py` script is used instead.

---

## Duplicates in the Corpus (Era E)

| Cluster | Forms present |
|---|---|
| SSH | SSH, ssh, sshd, OpenSSH, openssh |
| Port numbers | Port 22, port 22, Port 25, port 25, Port 80, port 80, same pattern for 143, 389, 443, 445, 2049, 53, 21 |
| Table names | filter Table, Filter Table, nat Table, NAT Table, mangle Table, Mangle Table, raw Table, Raw Table |
| Firewall variants | Perimeter Firewall, perimeter firewall |
| DMZ | DMZ, dmz, demilitarized zone |
| Multi-word case | Network Firewall, network firewall, Host Firewall, host firewall |
| Plural | Firewall / Firewalls, Stateful Firewall / Stateful Firewalls |

---

## Pipeline Design

The deployed script implements the following stages:

### Stage 1 — Rule-based normalization (auto-merge)

- **1a. Case-fold:** lowercase every name; group identical lowercases.
- **1b. Token reorder:** sort tokens alphabetically; group.
- **Acronym-aware singularization:** tokens that are fully uppercase or contain digits are left verbatim (`HTTPS`, `DDR4` are not singularized). Regular tokens ending in `ies` become `y`; a trailing `s` is stripped otherwise. This prevents `HTTP → HTTPS` merges from the singularizer confusing acronyms.

### Stage 1.5 — Morphological equivalence

- Detects `Prepare` / `Preparation Phase` / `Preparation Process` style equivalences.
- Requires a meta-token (`phase`, `stage`, `step`, `process`, `procedure`, `method`, `technique`, `approach`, `operation`) on at least one side.
- Strips meta-tokens, Porter-stems the remaining tokens, allows edit distance ≤ 2 with shared first 3 characters.

### Stage 1.75 — Explicit alias file (`--aliases aliases.json`)

- A JSON array of `[a, b]` pairs to merge unconditionally.
- Covers cases the rules and LLM both miss: `["CPU", "Processor"]`, `["Memory module", "RAM"]`, `["PC", "computer"]`, `["Fixed disk", "HDD"]`.
- Still subject to cannot-link (see Stage 3.5); aliases cannot chain into a bad merge.

### Stage 2 — Embedding similarity

- Compute cosine similarity of entity-name embeddings from the embedding server.
- `cos ≥ 0.97` (`--auto-cos`) auto-merges without LLM. Everything else in `[0.70, 0.97)` (`--gray-min` to `--auto-cos`) goes to the LLM.
- `cos < 0.70` auto-rejects.
- **Typo promotion:** pairs with the same first case-folded character and Levenshtein distance ≤ 2 are also promoted to gray even below `--gray-min` (so `Cleint` / `Client` gets an LLM look).

### Short-name rule (applies at every stage)

- Names of ≤3 alphanumeric characters are restricted. They can merge via:
  - Stage 1 (case-only) — i.e., already in the same normalized group.
  - Acronym-initials match (`DMZ` ↔ `Demilitarized Zone`) AND the long name has ≥ 2 tokens.
  - Mention evidence: the short name appears at least `--min-short-mentions` times in `kv_store_entity_chunks.json`.
- **Blocks:** `OM`, `OU`, `oy`, `MUL`, `Mil`, `dc`, `dccla` (junk tokens). Two short names together must be acronym matches.
- **Accepted cost:** `CPU` / `Processor`, `PC` / `computer` will not merge unless an alias is added.
- **The `dc → dccla` guard:** the acronym rule requires the long name to have ≥ 2 tokens. `dc` is a subsequence of `dccla`, but `dccla` is one token, so the rule rejects it.

### Stage 3 — Two-way LLM verification

- For each gray pair, ask the LLM twice: `A, B` and `B, A`. Both must say yes.
- `max_tokens=4`, `temperature=0`.
- Prompt: strict same-concept definition, generic few-shot examples (`engine / car → no`, `CPU / processor → yes`, `firewall / host firewall → no`, `error log / log file → no`).
- The two-order requirement costs ~2× LLM calls but catches asymmetries like `PCIe card ↔ PCIe` (first order yes, second no — only one false positive saved) and `Defrag ↔ TRIM` (second order caught a false yes — two-way check earned its keep).

### Stage 3.5 — Cannot-link-aware merge application

- All verdicts (accepts and rejects) are collected first.
- Accepts are applied strongest-first: Stage 1 → Stage 1.5 → alias → cosine descending.
- Before each union, check that no cross-component pair is in the reject set.
- If it would be, refuse the union and log at stage 3.5 with the conflicting pair named.
- **Why it matters:** DSU groups are transitive. One bad edge (e.g., `HDD ↔ SSD` at cosine 0.902) contaminates every merge chained onto it. Cannot-link stops this.

### Stage 4 — Apply merges to LightRAG

- For each component with >1 member, pick a **canonical target** and POST `/graph/entities/merge`.
- **Canonical choice:** the most-mentioned name (`kv_store_entity_chunks.json`). Ties broken by capitals count, then length, then name. Short junk tokens are deprioritised.
- Uses the same internal code path as `amerge_entities` — do not edit graph or vector files by hand.

---

## CLI Reference

```
--storage-dir          ./lightRAG/rag_storage
--lightrag-url         http://10.10.0.5:9621
--lightrag-api-key     <X-API-Key header>  # redacted; must be env var in production
--llama-url            http://10.10.0.5:9931/v1
--llama-api-key        <bearer token>      # redacted; must be env var in production
--llm-model            LightRAG-Logic
--embedding-model      LightRAG-Embedding
--log                  dedupe_decisions.jsonl
--dry-run              (flag)
--auto-cos             0.97
--gray-min             0.70
--min-short-mentions   2
--aliases              aliases.json
```

---

## Expected Output

`dedupe_decisions.jsonl` logs every decision (candidate, reject, apply, skip, merge). Stages logged: `1`, `1.5`, `1.75`, `2`, `3`, `3.5`, `4`, `apply`.

**Real-run result:** The script ran against `rag_storage/` with no dry-run flag. The log is too large to upload with these notes. The expected entity count reduction was previously estimated at ~24 absorbed entities into ~20 target groups; the actual deployed result would be in the log.

---

## Known Limitations

- **Single-character typos** in the middle of a word are caught by the Levenshtein promotion only when the first character matches and the string is >3 chars. Some typos still miss.
- **Multi-word reorderings** caught by Stage 1b require every token to appear; word order that carries meaning (rare in technical entity names) would be incorrectly merged.
- **PCIe card ↔ PCIe** — first LLM order yes, second no. Valid merge lost by two-way check. Alias file can restore it.
- **Fixed disk ↔ Hard disk** — subset guard (broader/narrower heuristic) blocks it. Alias file can restore it.

---

## The FP/FN Asymmetry

For entity resolution:

- **FP (false positive):** Merging two distinct concepts. Destroys graph structure — edges and chunk references absorbed into the wrong node. Hard to undo.
- **FN (false negative):** Leaving a duplicate. Recoverable with a second pass.

**FP is strictly worse than FN.** A model with higher accuracy but more FPs is a worse choice.
