---
title: Llama.cpp (Gigabyte Laptop) Glossary
description: GPU-offload and power-management terms used by the llama.cpp-on-laptop pages, distinct from the CPU-only Local-AI-VM glossary.
tags:
  - llama.cpp
  - gigabyte-gaming-a16
  - glossary
---

Terms that are specific to running llama.cpp with GPU acceleration on this laptop (GPU offload, VRAM management, power profiles). General llama.cpp/LLM vocabulary that applies across all Llama.cpp instances is defined once in [[Generic-Info/Llama.cpp/Llama-cpp-glossary|the canonical Llama.cpp glossary]] — link there instead of redefining it here.

---

## `-ngl` / `--n-gpu-layers`

Controls how many of a model's transformer layers are placed on the GPU versus left on CPU/system RAM. `0` keeps everything on CPU, a value at or above the model's total layer count (commonly `999` as a safe shorthand) offloads everything, and anything in between produces a hybrid split. On a GPU-enabled build, leaving `-ngl` unset does **not** mean "CPU only" the way it does on a CPU-only build — this laptop's build auto-fits as many layers as VRAM allows. See [[Gigabyte-LLama-Benchmarks#Partial offload — Qwen3-14B (too big for VRAM)|the Qwen3-14B test]] for a worked example, and always pass `-ngl` explicitly if you need to be sure of the split.

## VRAM

Video RAM — memory on the GPU itself, separate from system RAM. On this laptop the RTX 5060 Laptop GPU has 7705 MiB usable. Model weights and, by default, the KV cache both compete for this same pool once layers are offloaded (see `-nkvo` below). Unlike system RAM, VRAM doesn't quietly degrade into swap when it's exceeded — an over-budget allocation (e.g. too large a `-c` on top of a near-full offload) fails outright at load time.

## `-nkvo` / `--no-kv-offload`

Forces the KV cache to stay in system RAM even while model layers are offloaded to the GPU, instead of following the layers into VRAM by default. Frees up VRAM for more layers or a larger context, at the cost of an added PCIe round-trip per token. See [[Gigabyte-LLama-Benchmarks#`-nkvo` vs. context size|the `-nkvo` sweep]] for the measured crossover point on this laptop.

## `-fa` / `--flash-attn`

Enables flash attention, a GPU attention implementation that's typically both faster and more VRAM-efficient on supported cards. Not explicitly recorded as on or off during the Section 3 model sweep — see the open item on [[Gigabyte-LLama-Benchmarks|the benchmarks page]].

## Power profile (`powerprofilesctl`)

Linux's `power-profiles-daemon`, controlled via `powerprofilesctl set <performance|balanced|power-saver>`, primarily governs CPU clock/governor behavior. On this laptop it has a large, counterintuitive knock-on effect on GPU power headroom for GPU-bound workloads — see [[Power-profiles-and-gpu-performance|the hardware-bucket page on this]] for the underlying cause, and [[Gigabyte-LLama-Benchmarks|Benchmarks]] for the llama.cpp-specific throughput numbers.

## CUDA backend (`libggml-cuda.so`)

The GPU compute backend llama.cpp loads when a CUDA-capable NVIDIA GPU is detected, alongside the CPU backend (`libggml-cpu-alderlake.so` on this laptop's Alder Lake-based i7-13620H). Note that `llama-bench`'s `backend` column reports `CUDA` as soon as this backend is loaded, even at `-ngl 0` where no actual compute happens on the GPU — it reflects what loaded, not what's doing the work. Verify actual GPU utilization independently (e.g. `btop`) when it matters.

## mmproj / `--mmproj`

A multimodal projector file required for vision models to accept images. llama.cpp's router only treats a model as multimodal when the model lives in a subfolder containing the model `.gguf` and a second file whose name starts with `mmproj`. The projector must match the model weights; a mismatched projector fails to load or produces garbage output. See [[Local AI Model Presets and Context Tuning (Laptop)#Vision Projectors|the vision projector section]] for the specific projectors used on this laptop.

## Context Tiers

Per-model context size configurations organized into three tiers: Small (1/3 of maximum), Medium (2/3 of maximum), and Max (full usable value). The Max tier keeps the KV cache in VRAM for optimal performance, while smaller tiers allow more headroom for a second model when using `--models-max 2`. See [[Local AI Model Presets and Context Tuning (Laptop)#Context Tiers|the context tiers table]] for the specific values used on this laptop.

### Context Tier Examples

| Model | Binding limit | Small | Medium | Max |
|---|---|---:|---:|---:|
| Gemma 3 1B | Trained context (32K) | 10,922 | 21,845 | 32,768 |
| Qwen3.5-4B (both modes) | VRAM | 60,074 | 120,149 | 180,224 |
| Qwen3.5-9B (both modes) | VRAM | 24,917 | 49,834 | 74,752 |
| Qwen3.5-9B Uncensored (both modes) | VRAM | 25,770 | 51,541 | 77,312 |
| Gemma 4 E4B (both modes) | Trained context (128K) | 43,690 | 87,381 | 131,072 |

## KV Cache

The key-value cache used during autoregressive generation. By default it sits in VRAM next to the model weights, so a large context competes with the model for the same memory. When the cache does not fit, the system either offloads fewer layers to GPU (with fit enabled), fails to load, or moves the cache to system RAM using `--no-kv-offload`. The Max context tiers keep the cache in VRAM, while smaller tiers allow the cache to spill to CPU RAM when a second model is loaded.

### KV Cache Performance Impact

Moving a full KV cache from VRAM to system RAM is expensive. Benchmarks on this laptop show:

- **Gemma 4 E4B:** 18x generation slowdown (32.4 t/s in VRAM → 1.9 t/s in RAM)
- **Qwen3.5-9B:** 6.2x generation slowdown (34.8 t/s in VRAM → 5.6 t/s in RAM)
- **Qwen3.5-4B:** 10x generation slowdown (27.9 t/s in VRAM → 2.8 t/s in RAM)

This is why the Max context tiers keep `no-kv-offload` off. See [[Local AI Model Presets and Context Tuning (Laptop)#VRAM versus RAM Cache Benchmark|the VRAM versus RAM benchmark]] for full results.
