---
title: Local AI Model Presets and Context Tuning (Laptop)
description: Model cleanup, router presets, and per-model context tier configuration for llama.cpp on the Gigabyte Gaming A16 laptop.
tags:
  - llama.cpp
  - gigabyte-gaming-a16
  - presets
  - context
---

## Overview

This guide documents the process of cleaning up the model library on the Gigabyte Gaming A16 laptop, moving everything into a single `llama-server` router with a preset file, and measuring how much context each model can hold in VRAM. The laptop runs a different configuration than the OptiPlex CPU-only box—do not mix numbers between the two.

The final preset file is public: [presets.ini](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Daily%20Driver%20Laptop/Local%20AI/presets.ini)

---

## Hardware and Software

| Item | Detail |
|---|---|
| Machine | Gigabyte GAMING A16 laptop, CachyOS |
| CPU | Intel Core i7-13620H (6 P-cores + 4 E-cores) |
| GPU | NVIDIA RTX 5060 Laptop, about 7.7 GB usable VRAM |
| Display GPU | Intel iGPU (so the RTX is mostly free of desktop use) |
| RAM | about 31 GB |
| llama.cpp | Arch/CachyOS prebuilt package, 0.9.0-dev (build 12110, commit cd959bae99), GCC 16.2.1 |

The package is a prebuilt binary, not a self-compiled one, and it reports an asserts-enabled warning (explained in the earlier laptop log: the Arch package is built with CMAKE_BUILD_TYPE=None). All numbers below came from this same binary, so comparisons between runs are valid; treat absolute speeds as a floor rather than a ceiling.

Original server command:

```bash
llama-server --host 127.0.0.1,10.10.0.11 --port 9931 --no-ui --models-max 1 \
  --sleep-idle-seconds 180 --models-dir %h/Projects/LLMs/models \
  --models-preset %h/Projects/LLMs/presets.ini --api-key ""
```

Target command after this work: `--models-max 2` and no `--models-dir`, so only deliberately defined presets show up in `/v1/models`.

---

## Model Cleanup

### Kept Models

| Model | Role |
|---|---|
| Qwen3.5-9B Q4_K_M (+ mmproj) | Main general model, wiki jobs |
| Qwen3.5-9B Uncensored, HauhauCS Aggressive Q4_K_M | Uncensored general model |
| Qwen3.5-4B Q4_K_M (+ mmproj) | Smaller general model, LightRAG logic role |
| Gemma 4 E4B (plain, + mmproj) | Characters and general chat |
| Gemma 4 E4B Uncensored, TrevorJS Q4_K_M | Uncensored characters |
| Gemma 3 1B | Open WebUI background tasks (titles, tags, follow-ups) |
| Qwen2.5-VL-3B-Instruct (+ mmproj) | LightRAG vision role |
| Qwen3-Embedding-0.6B f16 | LightRAG embeddings |

### Removed Models and Reasons

| Model | Reason |
|---|---|
| Mistral-7B-Instruct v0.3 | Old and outclassed by the Qwen3.5 and Gemma 4 models |
| Gemma 3 12B | Failed to load fully onto the GPU with -ngl 999; it had only been kept to test how context-heavy the Hermes agent was |
| Qwen3-14B | Same failure to fit on the GPU |
| Qwen3-Embedding 4B and 8B | Different embedding dimension from the existing LightRAG index (1024), so unusable without re-indexing |
| Phi-4-mini-reasoning, DeepSeek-R1-Distill-Llama-8B | Never used; Qwen3.5 with thinking on covers the same ground |
| Gemma 3 4B | Kept originally because Gemma 4 E4B was a thinking model; once thinking could be turned off per preset it was redundant |
| Qwen3-8B abliterated, Qwen3.5-4B Uncensored (HauhauCS) | Replaced by the Qwen3.5-9B Uncensored and Gemma 4 E4B Uncensored |
| Llama 3.1 8B, Ministral-3 8B | Unused and overlapping with the Qwen3.5 / Gemma 4 models. Llama 3.1 8B's benchmark numbers remain in the OptiPlex log |
| Underdog-Saluki 27B IQ2-mix | Too large for the GPU at 2-bit, so it would run mostly on CPU with heavy quality loss |

### Uncensored Models

Newer uncensored releases were searched for. The decision was to replace the old Qwen3.5-4B uncensored model with the 9B version from the same author, and to add a Gemma 4 E4B uncensored model for character use.

- HauhauCS Qwen3.5-9B Aggressive: [Hugging Face](https://huggingface.co/HauhauCS/Qwen3.5-9B-Uncensored-HauhauCS-Aggressive). A third-party comparison reported little capability loss relative to the base model.
- TrevorJS Gemma 4 E4B uncensored: [Hugging Face](https://huggingface.co/TrevorJS/gemma-4-E4B-it-uncensored-GGUF), with the method described in [gemma-4-abliteration](https://github.com/TrevorS/gemma-4-abliteration). It reported a very low refusal rate with small drift from the original.

Neither was compared head-to-head against the base models on this hardware.

---

## Router Mode, Presets, and GPU Offload

### Router Mode with a Preset File Only

Without `--models-dir`, every model needs an explicit `model =` path, and vision models need an explicit `mmproj =` path. This fixes two problems seen earlier: projector files showing up as bogus models, and Open WebUI listing every GGUF in the folder.

Preset section names are the model names that appear in `GET /v1/models` and in Open WebUI. A preset can contain any server flag, written without the leading dashes.

### n-gpu-layers is Not Set

Earlier, a global `n-gpu-layers = 999` forced every layer onto the GPU, which is why the 12B and 14B models failed to load. The `llama-server --help` output on this build shows the relevant defaults:

- `-ngl` defaults to `auto`.
- `--fit` is on by default and adjusts unset arguments to fit in device memory.
- `--fit-target` defaults to a 1024 MiB margin per device.
- `--fit-ctx` defaults to a 4096-token minimum context.

Two consequences shaped the presets:

1. Fit only adjusts values that are not set. Setting `ctx-size` explicitly on every entry means fit trims GPU layers to make room instead of shrinking the context.
2. The margin was lowered to 512 MiB (`fit-target = 512` in the `[*]` block) because the display runs on the iGPU. If CUDA out-of-memory errors appear mid-chat, remove that line.

`threads = 6` is also in the defaults, matching the 6 P-cores. It only matters for layers that spill to the CPU.

### Two Models Loaded at Once

`--models-max 2` lets two models stay loaded. Each model is its own process and fit checks free VRAM when it loads, so a second model fits itself into what the first left behind. A router flag, `--models-memory-margin N`, unloads least-recently-used models until a new one fits; it is available in this build but is optional and was not required for the tests below.

---

## Thinking versus Non-Thinking

Goal: avoid having two copies of every model.

What was tested in Open WebUI:

- A per-request `reasoning-budget` of 0 in the advanced parameters disabled thinking on Gemma 4 E4B.
- The same setting did not disable thinking on the Qwen3.5-4B thinking preset. The apparent earlier success on the non-thinking Qwen entry was explained by the preset already setting thinking off on the server side.
- Adding `enable_thinking: false` as a custom parameter also failed for Qwen3.5.

Decision: control thinking in the preset, not per request. Each thinking-capable model has two entries pointing at the same file:

- Non-thinking: `reasoning = off` and `reasoning-budget = 0`.
- Thinking: neither key is set, so the template default (thinking on) applies.

Gemma 4 E4B was later split the same way. Loading both variants of the same file at once loads the weights twice, but in practice only one is used at a time.

---

## Vision Projectors

A vision model needs a separate `mmproj` file that matches it. With a preset file, the projector is named explicitly.

| Model | Projector |
|---|---|
| Qwen3.5-9B | Qwen3.5-9B-mmproj-Bf16.gguf |
| Qwen3.5-9B Uncensored | Reuses the 9B projector |
| Qwen3.5-4B | Qwen3.5-4B-mmproj-BF16.gguf |
| Gemma 4 E4B | gemma-4-E4B-it-mmproj-BF16.gguf |
| Gemma 4 E4B Uncensored | Reuses the plain E4B projector (the TrevorJS repo ships none) |
| Qwen2.5-VL-3B | Qwen2.5-VL-3B-Instruct-mmproj-Q8_0.gguf |
| Gemma 3 1B, Qwen3-Embedding | None (not vision models) |

Notes:

- The uncensored variants reusing the base projectors is an assumption: abliteration edits the language weights, not the vision encoder. Everything loaded and ran, but image-answer quality on the two uncensored models was not separately evaluated.
- Each projector costs VRAM (about 0.65 GB for the Qwen3.5-4B, about 0.9 GB for the 9B and Gemma). The Qwen3.5-9B with vision loaded used about 7.2 GB of the card, which is why its context ceiling is the lowest of the set.
- Files were renamed to `<model>-mmproj-<quant>.gguf` so each projector sits next to its model.

---

## Per-Model Tuning Flags

Flags were chosen from the `llama-server --help` output of this build.

| Setting | Applied to | Reason |
|---|---|---|
| `flash-attn = on`, `cache-type-k/v = q8_0` | Qwen3.5 models | Roughly halves KV memory; quantized V cache needs flash attention |
| KV cache left at f16, flash-attn auto | Gemma 4 | Not verified that Gemma 4's attention layout supports quantized KV on this build |
| `lazy-mode = on` | Gemma 4 | Reads the large per-layer-embedding tensors from disk on demand instead of keeping them resident (requires mmap) |
| `ubatch-size = 1024` | Qwen3.5-4B, Gemma 4 | Faster prompt processing where there is VRAM to spare |
| Default `ubatch-size` (512) | Qwen3.5-9B | Compute buffers compete with the weights for VRAM |
| `image-max-tokens = 2048`, `temp = 0.1` | Qwen2.5-VL (LightRAG vision) | Bounds time and context per image; low temperature makes descriptions more repeatable between runs |

Sampler defaults come from the model authors: Qwen3.5 uses its official thinking set (temp 0.6, top-p 0.95, top-k 20, min-p 0) and non-thinking set (temp 0.7, top-p 0.8, top-k 20, min-p 0); Gemma 4 uses temp 1.0, top-p 0.95, top-k 64. The Qwen values are the 9B card's and were applied to the 4B as well, which is an assumption. The low-temperature sampler lines for Gemma 3 1B (background tasks) are present but commented out.

Speculative decoding (`--spec-type`, which includes `ngram-mod` and `draft-mtp` in this build) was deliberately not enabled, because it is unknown whether it works with Qwen3.5's hybrid layers. It is a candidate for a later experiment on rewrite-style jobs.

---

## The KV Cache and VRAM

The KV cache is reserved at load time for the full configured `ctx-size`, whether or not a conversation fills it. By default it sits in VRAM next to the weights, so a large context competes with the model for the same memory.

When the cache does not fit:

1. **Fit on, explicit `ctx-size`:** fit keeps the context and offloads fewer layers to the GPU. The remainder runs on the CPU and generation slows.
2. **No room even with zero GPU layers:** the load fails.
3. **`no-kv-offload = true`:** the cache goes in system RAM, all weights stay on the GPU, and every token pays a PCIe transfer to read the cache.

An earlier sweep on Llama 3.1 8B (RTX 5060, `-nkvo` on and off across context sizes) found generation with the cache in RAM stayed flat at roughly 27 t/s while the in-VRAM case collapsed from 37.5 to 10.5 t/s by 72K context. That result suggested RAM-resident cache was a cheap escape hatch. The benchmark in section 10 shows that this does not hold when the cache is actually full (see section 12).

---

## Finding the Maximum Context per Model

Rather than guess context sizes, the largest context that stays fully in VRAM was measured per preset, using [preset_ctx_max_fit.sh](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/Benchmark%20Scripts/preset_ctx_max_fit.sh). Results: [max-laptop-ctx-results.tsv](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/max-laptop-ctx-results.tsv).

The columns are the model, the context configured at the time, the maximum context that fit in VRAM, the recommended value, and the number of trials the script needed. The recommended value equalled the measured maximum for every model. The measurements depend on the settings active at the time (q8_0 KV for Qwen, f16 KV for Gemma, projector loaded, `fit-target = 512`); changing any of them invalidates the number.

| Model | Configured before | Max context in VRAM | Trials |
|---|---|---|---|
| Gemma 3 1B | 16,384 | 262,144 | 10 |
| Qwen3.5-4B (and thinking) | 32,768 / 131,072 | 180,224 | 18 |
| Qwen3.5-9B (and thinking) | 32,768 / 65,536 | 74,752 | 16 |
| Qwen3.5-9B Uncensored (and thinking) | 32,768 / 65,536 | 77,312 | 16 |
| Gemma 4 E4B | 65,536 | 185,344 | 18 |
| Gemma 4 E4B Uncensored | 65,536 | 185,856 | 18 |

### Capping at the Trained Context

Two models fit more context in VRAM than they were trained to handle:

- **Gemma 3 1B:** measured 262,144 in VRAM, but the [Gemma 3 model card](https://ai.google.dev/gemma/docs/core/model_card_3) lists 32K tokens for the 1B size.
- **Gemma 4 E4B (and its uncensored fine-tune):** measured about 185K in VRAM, but the [Gemma 4 model card](https://ai.google.dev/gemma/docs/core/model_card_4) lists 128K tokens for E2B and E4B.

Their Max tiers use the trained size (32,768 and 131,072), and the Small and Medium tiers were recalculated from that. The Qwen3.5 maxima (180,224 for the 4B, 74,752 and 77,312 for the 9B models) are below Qwen3.5's native context, which is understood to be 262,144 but was not re-verified here. `n_ctx_train` in each model's load log shows what llama.cpp itself reads from the file.

### Context Tiers

Each tier is rounded down: Small = 1/3, Medium = 2/3, Max = the full usable value.

| Model | Binding limit | Small | Medium | Max |
|---|---|---:|---:|---:|
| Gemma 3 1B | Trained context (32K) | 10,922 | 21,845 | 32,768 |
| Qwen3.5-4B (both modes) | VRAM | 60,074 | 120,149 | 180,224 |
| Qwen3.5-9B (both modes) | VRAM | 24,917 | 49,834 | 74,752 |
| Qwen3.5-9B Uncensored (both modes) | VRAM | 25,770 | 51,541 | 77,312 |
| Gemma 4 E4B (both modes) | Trained context (128K) | 43,690 | 87,381 | 131,072 |
| Gemma 4 E4B Uncensored (both modes) | Trained context (128K) | 43,690 | 87,381 | 131,072 |

The idea is that the Max tier is a good default for most use, and the smaller tiers exist for when VRAM is needed by a second model.

---

## VRAM versus RAM Cache Benchmark

[bench-max-ctx.sh](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/Benchmark%20Scripts/bench-max-ctx.sh) ran each preset at its maximum context, once with the KV cache in VRAM (`nkvo = 0`) and once in RAM (`nkvo = 1`). The `depth` column in the results shows the cache was filled to within 640 tokens of the full context (the pp512 + tg128 test length), so these numbers reflect attention over a nearly full cache. Results: [bench-laptop-results.tsv](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/bench-laptop-results.tsv).

| Model | Context | Gen, cache in VRAM (t/s) | Gen, cache in RAM (t/s) | Gen slowdown | Prompt, VRAM (t/s) | Prompt, RAM (t/s) | Prompt slowdown |
|---|---|---:|---:|---:|---:|---:|---:|
| Gemma 3 1B | 262,144 | 107.2 | 8.7 | 12x | 3,327 | 1,935 | 1.7x |
| Qwen3.5-4B | 180,224 | 27.9 | 2.8 | 10x | 584 | 407 | 1.4x |
| Qwen3.5-4B thinking | 180,224 | 28.0 | 2.9 | 10x | 557 | 423 | 1.3x |
| Qwen3.5-9B | 74,752 | 34.8 | 5.6 | 6.2x | 991 | 704 | 1.4x |
| Qwen3.5-9B thinking | 74,752 | 34.7 | 5.6 | 6.1x | 997 | 708 | 1.4x |
| Qwen3.5-9B Uncensored | 77,312 | 34.8 | 5.4 | 6.4x | 962 | 654 | 1.5x |
| Qwen3.5-9B Uncensored thinking | 77,312 | 33.4 | 5.4 | 6.2x | 945 | 688 | 1.4x |
| Gemma 4 E4B | 185,344 | 33.7 | 1.9 | 18x | 1,061 | 519 | 2.0x |
| Gemma 4 E4B Uncensored | 185,856 | 32.4 | 1.9 | 17x | 1,084 | 497 | 2.2x |

The benchmark ran before the trained-context cap, so the Gemma 3 1B and Gemma 4 rows were measured at contexts (262,144 and about 185K) above what their final Max tiers use. The Qwen3.5 rows are unaffected. The capped Gemma entries hold a smaller cache, so the VRAM-versus-RAM gap should be smaller than shown, but that was not re-measured.

Findings:

- **With the cache in VRAM, every model stays usable at its maximum context** (roughly 28 to 107 t/s generation).
- **Moving a full cache to RAM is expensive.** Generation drops 6x to 18x, and prompt processing drops 1.3x to 2.2x. Gemma 4 suffers most and the 9B least. Likely contributors, not tested: Gemma's cache is f16 while Qwen's is q8_0, and Qwen3.5 has only a fraction of layers using full attention.
- **Thinking and non-thinking variants perform alike**, as expected for the same weights.
- **The Max tiers should keep `no-kv-offload` off.** That is how the presets are set.

---

## Preset Layout

Naming: `<Model>-<Mode>-<Tier>`, for example `Qwen3.5-9B-Thinking-Medium`, `Gemma-4-E4B-Uncensored-Max`, `gemma-3-1b-Small`. Non-thinking is the default and has no mode word. The Gemma 3 1B entries have no thinking variant.

Job presets sit at the end of the file:

| Preset | Model | Context | Notes |
|---|---|---|---|
| `LightRAG-Embedding` | Qwen3-Embedding-0.6B f16 | 8,192 | `embedding = true`, batch and ubatch 4096 (embedding input must fit one physical batch), `cache-ram = 0` |
| `LightRAG-Logic` | Qwen3.5-4B, non-thinking | 24,576 | Extraction and answer role; q8_0 KV; projector loaded |
| `LightRAG-Vision` | Qwen2.5-VL-3B | 16,384 | `image-max-tokens = 2048`, `temp = 0.1` |
| `Quartz-Wiki-Rewrite` | Qwen3.5-9B | 256,000 | `no-kv-offload = true`, thinking sampler set |
| `Quartz-Wiki-New-Content` | Qwen3.5-9B | 74,752 | Matches the 9B Max tier, so the cache stays in VRAM |

---

## Corrections Made Along the Way

- **Default `-ngl`:** the earlier glossary said it defaulted to 0 on GPU builds. The `--help` output for this build shows `auto` with fit on by default.
- **Expected `no-kv-offload` penalty:** it was first estimated at roughly 10% to 25% from the Llama 8B sweep, then guessed to be smaller on Qwen3.5. The depth-filled benchmark shows 6x to 18x generation slowdowns at maximum context. The earlier sweep most likely measured an allocated-but-empty cache, which would explain its flat RAM-cache curve; this is an inference, since the sweep's exact method was not re-checked.
- **Maximum context versus trained context:** the first set of tiers was computed from what fits in VRAM alone, which put Gemma 3 1B and Gemma 4 E4B above their trained lengths. Both are now capped at the trained size and their Small and Medium tiers recalculated.
- **Per-request thinking control:** assumed possible for all models; it works for Gemma 4 only in the tests run here.

---

## Caveats and Open Items

- **`Quartz-Wiki-Rewrite` is slow by design.** It needs to modify about 260 files in one pass, so the cache stays in RAM at 256,000 tokens. That is more than three times the depth of the 9B's 74K result, which already ran at 5.6 t/s, so expect no better than that. Its speed and prompt-processing time at that depth were not measured, and optimizing it is deferred.
- **Fit at less than Max tiers.** The context ceilings assume one model loaded. With `--models-max 2`, a second large preset will have fit trim its GPU layers.
- **LightRAG's three models cannot all fit in VRAM at the same time** (embedding, logic, vision). With `--models-max 2`, expect reloads or CPU spill when it alternates between them. This was not measured.
- **`LightRAG-Logic` loads a vision projector** it may not need for text-only extraction, which costs about 0.65 GB of VRAM. Remove the `mmproj` line if headroom matters there.
- **Image quality on the uncensored models** (shared projectors) was not evaluated.
- **Not yet tried:** `--spec-type` speculative decoding, `--models-memory-margin`, and a CPU thread-count sweep for hybrid P/E cores.
- **Benchmark scripts** are linked for reference; their exact search method is described in the scripts themselves and was not reviewed for this write-up.

---

## Files

| File | Link |
|---|---|
| Presets | [presets.ini](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Daily%20Driver%20Laptop/Local%20AI/presets.ini) |
| Max-context results | [max-laptop-ctx-results.tsv](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/max-laptop-ctx-results.tsv) |
| VRAM vs RAM benchmark results | [bench-laptop-results.tsv](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/bench-laptop-results.tsv) |
| Max-context script | [preset_ctx_max_fit.sh](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/Benchmark%20Scripts/preset_ctx_max_fit.sh) |
| Benchmark script | [bench-max-ctx.sh](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/Benchmark%20Scripts/bench-max-ctx.sh) |

---

![[bench-laptop-results.tsv]]
*VRAM versus RAM cache benchmark results showing generation slowdown when moving full KV cache to system RAM.*
