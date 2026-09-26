---
title: Installing Llama.cpp on the Gigabyte Laptop
description: Installing llama.cpp with CUDA support via CachyOS/Arch packages, and the GPU flags used for offload tuning.
tags:
  - llama.cpp
  - gigabyte-gaming-a16
  - cuda
---

## Installation

I installed llama.cpp on this laptop as prebuilt CachyOS/Arch repository packages rather than compiling from source: `llama-cpp` for the llama.cpp binaries themselves, and `ggml-cuda` for the CUDA backend, both via `pacman`.

This is a different install path from the [[Homelab/Platform/Local-AI-VM/Llama.cpp/Build-and-cpu-tuning|Local-AI-VM's self-compiled build]] — that VM is a separate, CPU-only machine, and its `-march=native` / stale-incremental-build troubleshooting doesn't apply here, since there's no local `build/` directory on this laptop to have gone stale.

## The "asserts enabled" warning — root cause confirmed

Every `llama-bench` run on this laptop prints `warning: asserts enabled, performance may be affected`. Unlike the Local-AI-VM's version of this same warning (caused by a stale incremental build linking a debug-flavored object file — see [[Homelab/Platform/Local-AI-VM/Llama.cpp/Build-and-cpu-tuning#Gotchas|its Gotchas]]), that explanation doesn't hold here, since this is a fresh distro package with no incremental-build history.

**✅ Root cause confirmed (2026-09-25)** by reading the actual Arch `PKGBUILD` recipes for `ggml` and `llama-cpp`:

- The **`ggml`** package (supplies `libggml-cuda.so` etc.) is built with `-DCMAKE_BUILD_TYPE=Release` — correctly optimized. Its PKGBUILD sets `options=(!buildflags)`, which just skips `makepkg`'s own default hardening `CFLAGS`/`CXXFLAGS`/`LDFLAGS` injection — a packaging choice unrelated to asserts/debug behavior, since build type is what actually gates that.
- The **`llama-cpp`** package itself is the actual culprit: its PKGBUILD explicitly sets `-DCMAKE_BUILD_TYPE=None` — not `Release`, not `Debug`, no build type at all. This is a deliberate maintainer choice in the upstream Arch package, not a stale-build artifact or an accident. llama.cpp's own `CMakeLists.txt` gates asserts/debug-oriented code paths on the build type being `Release`; `None` doesn't satisfy that check, so asserts stay compiled in.
- **This confirms the warning is real, not a false alarm** — the packaged `llama-cpp` binary genuinely isn't a fully optimized Release build, by the Arch maintainer's own explicit configuration. But since `ggml` (which does the actual heavy tensor math, including the CUDA/CPU backends) *is* correctly built as `Release`, the compute-heavy inner loops are still optimized — `None` mainly affects the thinner `llama-cpp` glue/CLI code, not `ggml`'s core kernels. This likely explains why the packaged binary is still usably fast despite the warning, though it's not a guaranteed fully-optimized build end-to-end.

**Practical takeaway:** the "asserts enabled" warning on this laptop is an explained upstream packaging choice (`CMAKE_BUILD_TYPE=None` in the `llama-cpp` PKGBUILD), not a broken installation. For a guaranteed fully-optimized build, self-compile from source with `-DCMAKE_BUILD_TYPE=Release` (same process as the [[Homelab/Platform/Local-AI-VM/Llama.cpp/Build-and-cpu-tuning|Local-AI-VM's build]]) instead of relying on the distro package.

**What this means for the numbers on this page:** the power-profile comparisons on [[Benchmarks|the Benchmarks page]] remain valid regardless, since every run used the same package binary throughout — profile-to-profile comparisons are unaffected. Only the *absolute* t/s numbers should be treated as a floor rather than a ceiling, until/unless a self-compiled Release build is tested for direct comparison.

## GPU-specific flags

These flags matter specifically for GPU offload and aren't relevant on the CPU-only Local-AI-VM:

| Flag | Meaning | Default if unset |
|---|---|---|
| [[Llama-cpp-glossary#`-ngl` / `--n-gpu-layers`\|`-ngl, --n-gpu-layers <N>`]] | Layers to offload to GPU | `0` on CPU-only builds; auto-fits to VRAM on this GPU-enabled build if left unset — always pass explicitly to be sure |
| [[Llama-cpp-glossary#`-nkvo` / `--no-kv-offload`\|`-nkvo, --no-kv-offload`]] | Keep KV cache in system RAM even when layers are offloaded | KV cache follows offloaded layers to VRAM |
| [[Llama-cpp-glossary#`-fa` / `--flash-attn`\|`-fa, --flash-attn`]] | Enable flash attention | disabled |
| `-mg, --main-gpu <N>` | Which GPU index is "main" | `0` (only matters with multiple GPUs — not relevant on this single-GPU laptop) |
| `-sm, --split-mode <none\|layer\|row>` | How to split work across multiple GPUs | `layer` (not relevant here) |
| `-ts, --tensor-split <a,b,...>` | Ratio to split layers across multiple GPUs | none (not relevant here) |

**Note:** requesting more layers via `-ngl` than the model has doesn't error — it silently clamps to full offload. Always confirm actual offload via the model-load log line (`offloaded X/X layers to GPU`) rather than trusting the flag value alone.

## Benchmark and sweep scripts

Model sweeps on this laptop were run with the same two scripts used on the Local-AI-VM box, for a like-for-like comparison point:

- [Model-Benchmark.sh](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/Benchmark%20Scripts/Model-Benchmark.sh) — per-model `llama-bench` pp512/tg128 sweep.
- [ctx_sweep.sh](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/Benchmark%20Scripts/ctx_sweep.sh) — doubles `-c` per model to find the max safe context, adapted here for VRAM instead of system RAM.

See [[Benchmarks|Benchmarks]] for results.
