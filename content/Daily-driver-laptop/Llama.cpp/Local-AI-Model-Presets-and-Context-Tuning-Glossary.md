---
title: Local AI Model Presets Glossary (Laptop)
description: Terms specific to the llama.cpp model presets and context configuration on the Gigabyte Gaming A16 laptop.
tags:
  - llama.cpp
  - gigabyte-gaming-a16
  - presets
  - glossary
---

Terms that are specific to the model preset configuration and context tier management on this laptop. General llama.cpp/LLM vocabulary that applies across all Llama.cpp instances is defined once in [[Homelab/Platform/Local-AI-VM/Llama.cpp/Llama-cpp-glossary|the canonical Llama.cpp glossary]] — link there instead of redefining it here.

---

## Router Mode

llama-server's `--models-preset` mode where models are loaded on demand based on the `model` field in a request, rather than being discovered from a directory. Without `--models-dir`, every model needs an explicit `model =` path in the preset file. This mode provides better control over which models are available and prevents projector files from showing up as bogus models.

## Preset File

A `presets.ini` file that defines model configurations for llama-server. Each section corresponds to a model that can be loaded by the router. Sections can include any server flag written without leading dashes. Generic presets (one per model file) and role presets (for specific use cases like LightRAG) can coexist in the same file.

### Preset Section Naming

Section names must match the model names that appear in `GET /v1/models` and in Open WebUI exactly, including case sensitivity. For example, `[qwen2.5-vl-3b]` must match the exact name returned by the `/v1/models` endpoint.

## --models-max

A llama-server flag that caps how many models can be resident simultaneously. On a tight-RAM host, `--models-max 1` prevents two large models from competing for RAM but forces a load/unload cycle on role switches. `--models-max 2` allows two models to stay loaded, with fit checking free VRAM when a second model loads.

## fit

llama.cpp's automatic fitting feature that adjusts unset arguments to fit in device memory. When `fit = on` (the default), the server will trim GPU layers to make room for the configured context size rather than shrinking the context. This is important for controlling memory usage explicitly.

### fit-target

The memory margin (in MiB) that fit leaves on each device. The laptop's presets use `fit-target = 512` because the display runs on the iGPU. If CUDA out-of-memory errors appear mid-chat, remove that line.

## Thinking versus Non-Thinking Presets

Some models support both thinking (chain-of-thought) and non-thinking modes. Instead of keeping two separate model files, the laptop uses two preset sections pointing at the same file:

- **Non-thinking:** `reasoning = off` and `reasoning-budget = 0`
- **Thinking:** Neither key is set, so the template default (thinking on) applies

Loading both variants of the same file at once loads the weights twice, but in practice only one is used at a time.

## Vision Projector (mmproj)

A separate file required for multimodal models to accept images. llama.cpp's router only treats a model as multimodal when the model's preset includes an explicit `mmproj =` path pointing to the projector file. The projector must match the model weights; a mismatched projector fails to load or produces garbage output.

### Projector Naming Convention

Projectors are named `<model>-mmproj-<quant>.gguf` and stored in the same subfolder as their corresponding model file. For example, `Qwen3.5-9B-Q4_K_M.gguf` pairs with `Qwen3.5-9B-mmproj-Bf16.gguf`.

## Context Tiers

Per-model context size configurations organized into three tiers:

- **Small:** 1/3 of the maximum usable context
- **Medium:** 2/3 of the maximum usable context
- **Max:** The full usable value

The Max tier keeps the KV cache in VRAM for optimal performance, while smaller tiers allow more headroom for a second model when using `--models-max 2`.

### Binding Limit

The maximum context a model can actually use, determined by either:
- The model's trained context size (from the model card or `n_ctx_train` in the load log)
- The maximum context that fits in VRAM (measured empirically)

If the measured VRAM capacity exceeds the trained context, the trained context is used as the binding limit.

## KV Cache in VRAM versus RAM

The KV cache can reside either in GPU VRAM (default) or in system RAM (with `no-kv-offload = true`). Keeping the cache in VRAM provides optimal performance, but moving it to RAM frees up VRAM for more layers or a larger context.

### Performance Impact

Moving a full KV cache from VRAM to system RAM is expensive:

- **Gemma 4 E4B:** 18x generation slowdown
- **Qwen3.5-9B:** 6.2x generation slowdown
- **Qwen3.5-4B:** 10x generation slowdown

This is why the Max context tiers keep `no-kv-offload` off. See [[Local AI Model Presets and Context Tuning (Laptop)#VRAM versus RAM Cache Benchmark|the VRAM versus RAM benchmark]] for full results.

## Job Presets

Specialized preset configurations for specific use cases that sit at the end of the presets.ini file. Examples include:

- **LightRAG-Embedding:** Optimized for embedding tasks with small context and large batch sizes
- **LightRAG-Logic:** Optimized for extraction and reasoning with thinking disabled
- **LightRAG-Vision:** Optimized for vision analysis with image token limits
- **Quartz-Wiki-Rewrite:** A specialized preset for wiki rewriting jobs with large context in RAM

## Preset Layout

The organization of the presets.ini file follows this structure:

1. **[*] defaults:** Global defaults that apply to all sections unless overridden
2. **Generic model presets:** One section per model file on disk
3. **Role presets:** Sections for specific use cases (LightRAG roles, quiz agents, etc.)

This layout makes it easy to see all available models at a glance while also documenting specialized configurations.
