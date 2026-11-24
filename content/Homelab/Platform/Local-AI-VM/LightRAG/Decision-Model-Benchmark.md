---
title: Decision-Model Benchmark for Entity Merge
description: Benchmark of decision models for entity merge verification, including FP/FN asymmetry analysis and model selection.
tags:
  - LightRAG
  - homelab
  - fedora
  - rag
  - benchmark
---

## Overview

This page documents the decision-model benchmark for entity merge verification on the Local-AI-VM Fedora platform.

---

## The FP/FN Asymmetry

For entity resolution:

- **FP (false positive):** Merging two distinct concepts. Destroys graph structure — edges and chunk references absorbed into the wrong node. Hard to undo.
- **FN (false negative):** Leaving a duplicate. Recoverable with a second pass.

**FP is strictly worse than FN.** A model with higher accuracy but more FPs is a worse choice.

---

## Models Tested

| Model | Type | Size | Endpoint |
|---|---|---|---|
| Qwen3.5-4B thinking | Generative | 4B | 9931 (chat completions) |
| Qwen3.5-4B thinkless | Generative | 4B | 9933 (chat completions) |
| Qwen3.5-2B thinking | Generative | 2B | 9931 |
| Qwen3.5-2B thinkless | Generative | 2B | 9933 |
| Laya-Q8_0 | Decision | ~421M | 9931 `/v1/systemone` |
| d1-3B-Q4_K_M | Decision | 3B | 9931 `/v1/systemone` |
| strands-decider-2b | Decision | 2B | dropped (failed to load) |

---

## Run 1 — Broad Benchmark (125 pairs)

| Model | Correct | Wrong | Accuracy |
|---|---|---|---|
| qwen4b_thinkless | 123 | 2 | 0.98 |
| d1-3B | 117 | 8 | 0.94 |
| qwen2b_thinkless | 96 | 29 | 0.77 |
| Laya | 79 | 46 | 0.63 |
| qwen4b thinking | 25 | 0 | 0.20 (100 errors) |
| qwen2b thinking | 0 | 0 | 0.00 (125 errors) |

Thinking models: every response had `content: ""` and `finish_reason: length`. They exhausted `max_tokens` inside the reasoning block. **Thinking mode is incompatible with single-token output tasks.**

---

## Run 2 — Adversarial (110 pairs, amplifies each model's weakness)

| Model | Correct | Wrong | Acc | TP | TN | FP | FN |
|---|---|---|---|---|---|---|---|
| qwen4b_thinkless | 104 | 6 | 0.95 | 40 | 64 | 1 | 5 |
| d1-3B | 95 | 15 | 0.86 | 45 | 50 | 15 | 0 |
| qwen2b_thinkless | 87 | 23 | 0.79 | 28 | 59 | 6 | 17 |

**Selected: qwen4b_thinkless.** FP rate 1.8% vs d1's 26%. Its 5 FNs are all single-character typos, which the embedding gate partially catches.

---

## Lessons

- Report FP and FN separately. A single accuracy number hides the failure mode.
- Adversarial sets are more informative than representative ones. Run 2 decided the benchmark in less time.
- Decision models (Laya, d1) are not drop-in replacements for LLMs on conceptual tasks. Laya was near random; d1 was better but below a 4B generative model.
- Thinking mode is unusable for single-token output tasks.
