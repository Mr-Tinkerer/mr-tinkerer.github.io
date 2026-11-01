---
title: Llama.cpp Installation on Gigabyte Laptop
description: Installing llama.cpp with CUDA support via CachyOS/Arch packages and GPU offload tuning.
tags:
  - llama.cpp
  - gigabyte-gaming-a16
  - cuda
---

## Overview

This page documents llama.cpp installation on the Gigabyte Gaming A16 laptop, using prebuilt CachyOS/Arch packages with CUDA backend support.

## Installation

### Package Installation

llama.cpp is installed as prebuilt CachyOS/Arch repository packages rather than compiling from source:

- `llama-cpp` — llama.cpp binaries
- `ggml-cuda` — CUDA backend

Installation is performed via `pacman`.

### Installation Path Comparison

This installation differs from the [[Homelab/Platform/Local-AI-VM/Llama.cpp/Build-and-cpu-tuning|Local-AI-VM's self-compiled build]]:

- **Local-AI-VM:** CPU-only machine with `-march=native` compilation and stale-incremental-build troubleshooting
- **Gigabyte Laptop:** Prebuilt packages with no local build directory history

## Asserts Enabled Warning

### Warning Observation

Every `llama-bench` run prints: `warning: asserts enabled, performance may be affected`.

### Root Cause Analysis (2026-09-25)

Reading Arch `PKGBUILD` recipes for `ggml` and `llama-cpp` confirmed the warning's origin:

#### ggml Package

- Built with `-DCMAKE_BUILD_TYPE=Release` — correctly optimized
- PKGBUILD sets `options=(!buildflags)` — skips makepkg's default hardening CFLAGS/CXXFLAGS/LDFLAGS injection
- This packaging choice is unrelated to asserts/debug behavior

#### llama-cpp Package

- **Root cause:** PKGBUILD explicitly sets `-DCMAKE_BUILD_TYPE=None`
- Not `Release`, not `Debug` — no build type specified
- llama.cpp's `CMakeLists.txt` gates asserts on build type being `Release`; `None` fails this check
- **Conclusion:** This is a deliberate upstream Arch maintainer choice, not a stale-build artifact

### Practical Impact

- The `asserts enabled` warning is real and explained by upstream packaging
- **ggml is correctly optimized** — The heavy tensor math (including CUDA/CPU backends) uses Release build
- **llama-cpp glue code is not optimized** — The thinner CLI/glue code is affected by `None` build type
- **Performance:** Absolute t/s numbers should be treated as a floor rather than a ceiling

### Self-Compilation Alternative

For a guaranteed fully-optimized build, self-compile from source with `-DCMAKE_BUILD_TYPE=Release` (same process as the Local-AI-VM build).

## GPU-Specific Flags

These flags matter specifically for GPU offload and are not relevant on the CPU-only Local-AI-VM:

| Flag | Meaning | Default if unset |
|------|---------|------------------|
| `-ngl`, `--n-gpu-layers <N>` | Layers to offload to GPU | `0` on CPU-only; auto-fits to VRAM on GPU-enabled builds |
| `-nkvo`, `--no-kv-offload` | Keep KV cache in system RAM | KV cache follows offloaded layers to VRAM |
| `-fa`, `--flash-attn` | Enable flash attention | disabled |
| `-mg`, `--main-gpu <N>` | Which GPU index is "main" | `0` (only matters with multiple GPUs) |
| `-sm`, `--split-mode <none\|layer\|row>` | How to split work across multiple GPUs | `layer` |
| `-ts`, `--tensor-split <a,b,...>` | Ratio to split layers across multiple GPUs | none |

### Important Notes

- Requesting more `-ngl` layers than the model has silently clamps to full offload
- Always confirm actual offload via the model-load log line (`offloaded X/X layers to GPU`) rather than trusting the flag value alone

## Benchmark Scripts

Model sweeps use the same scripts as the Local-AI-VM for like-for-like comparison:

- [Model-Benchmark.sh](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/Benchmark%20Scripts/Model-Benchmark.sh) — Per-model `llama-bench` pp512/tg128 sweep
- [ctx_sweep.sh](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/Benchmark%20Scripts/ctx_sweep.sh) — Doubles `-c` per model to find max safe context, adapted for VRAM

See [[Benchmarks|Benchmarks]] for results.
