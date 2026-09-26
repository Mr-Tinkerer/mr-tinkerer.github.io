---
title: Llama.cpp Benchmarks (Gigabyte Laptop, RTX 5060)
description: GPU-offload throughput, power-profile, and -nkvo/context sweeps for llama.cpp on the Gigabyte GAMING A16.
tags:
  - llama.cpp
  - gigabyte-gaming-a16
  - benchmark
---

This machine (Gigabyte GAMING A16, RTX 5060 Laptop GPU, CachyOS) is separate hardware from the CPU-only Local-AI-VM tracked in [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks|the Local-AI-VM benchmarks]] — don't mix numbers between the two, though the model sweep below uses the same model set for comparison.

See [[Installation#GPU-specific flags|Installation]] for what each flag below means, and see [[Homelab/Platform/Local-AI-VM/Llama.cpp/Llama-cpp-glossary|the canonical glossary]] for `pp512`/`tg128` and other general terms.

---

## Full model sweep — full GPU offload (`-ngl 999`)

| Model | Params | pp512 (t/s) | tg128 (t/s) | Max safe `-c` |
|---|---|---|---|---|
| DeepSeek-R1-Distill-Qwen 1.5B | 1.5B | 8587.02 | 175.96 | 2,097,152 |
| Gemma 2 2B | 2B | 6024.78 | 100.19 | 262,144 |
| Llama 3.2 3B | 3B | 4436.13 | 164.79 | 262,144 |
| Qwen 2.5 3B | 3B | 4272.83 | 93.30 | 1,048,576 |
| Phi-3.5 Mini 3.8B | 3.8B | 3106.71 | 76.32 | 131,072 |
| Llama 3.1 8B | 8B | 1878.74 | 44.42 | 262,144 |
| Mistral 7B | ~7B | 1913.68 | 48.97 | 262,144 |

Max-safe-`-c` figures here are VRAM-driven ceilings (8GB card), not directly comparable to the RAM/OOM ceilings on the CPU-only VM even for the same model name.

**Open items:**
- Power profile in effect during this sweep wasn't recorded — given the [[Power-profiles-and-gpu-performance|power-profile finding]], confirm/record it and ideally re-run under the laptop's chosen standard profile.
- `-fa`/flash-attention state during this run wasn't recorded.
- Compare against the [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks|Local-AI-VM CPU baselines]] once profile/flash-attention state is confirmed, to quantify GPU speedup per model.

---

## Power profile vs. GPU-bound throughput — Llama 3.1 8B, `-ngl 999`

| Power Profile | pp512 (t/s) | tg128 (t/s) |
|---|---|---|
| `performance` | 1551.00 ± 212.77 | 31.72 ± 0.44 |
| `power-saver` | 2081.93 ± 213.33 | 59.98 ± 0.23 |

`power-saver` beat `performance` here by ~34% on pp512 and ~89% on tg128 — the opposite of what the profile names suggest. The root cause is a laptop hardware characteristic, not a llama.cpp one: see [[Power-profiles-and-gpu-performance|the Gigabyte laptop hardware page]] for the `nvidia-smi` power-limit numbers and explanation.

**Practical takeaway:** for GPU-bound inference (`-ngl` high/full offload) on this laptop, use `power-saver`.

## Power profile vs. CPU-only throughput — Llama 3.1 8B, `-ngl 0`

| Power Profile | pp512 (t/s) | tg128 (t/s) |
|---|---|---|
| `power-saver` | 470.88 ± 36.38 | 2.84 ± 0.02 |
| `performance` | 725.47 ± 2.73 | 7.29 ± 0.56 |

Here `performance` wins, as its name suggests — ~54% faster pp512, ~2.6x faster tg128. This confirms the profile is tuned around sustained CPU clocks: that's a win for CPU-bound work and a loss for GPU-bound work, per [[Power-profiles-and-gpu-performance|the hardware-level explanation]].

**Note:** setting `-ngl 0` does not stop llama.cpp from loading the CUDA backend — `llama-bench`'s `backend` column still reports `CUDA`, which reflects what loaded, not what computed. Confirmed via `btop` that the CPU (not GPU) did the work in these runs. `CUDA_VISIBLE_DEVICES=""` would suppress CUDA entirely for a cleaner label, if wanted for future runs.

**Combined rule for this machine:**
- GPU-bound (`-ngl` high/full) → `power-saver`.
- CPU-only (`-ngl 0`) → `performance`.
- Picking the wrong profile is a bigger loss (up to ~2.6x) than most other single tuning choice tested on this machine so far.

**Open items:**
- `balanced` profile not yet tested for the CPU-only case.
- Thread count (`-t`) wasn't explicitly set for these runs — worth a sweep given the hybrid P/E-core CPU (12+4), to see if the P/E split changes where diminishing returns kick in compared to a uniform-core CPU.

---

## Full GPU offload vs. CPU-only, by mode — Llama 3.1 8B Q4_K_M

| Mode | pp512 (t/s) | tg128 (t/s) |
|---|---|---|
| Full GPU (`-ngl 999`) | 1707.84 ± 26.98 | 39.42 ± 0.22 |
| CPU-only (`-ngl 0`), `power-saver` | 470.88 ± 36.38 | 2.84 ± 0.02 |
| CPU-only (`-ngl 0`), `performance` | 725.47 ± 2.73 | 7.29 ± 0.56 |

---

## `-nkvo` vs. context size — Llama 3.1 8B, `-ngl 999`

**Purpose:** whether an oversized KV cache (relative to remaining VRAM after weights) causes a hard cutover to CPU-speed generation, or a gradual slowdown — and where `-nkvo` becomes worth using on this GPU.

| Context (`-c`) | Without `-nkvo` (t/s) | With `-nkvo` (t/s) |
|---|---|---|
| ~11,264 | 37.5 | 33.2 |
| 23,552 | 22.2 | 27.7 |
| 29,696 | 17.6 | 27.0 |
| 41,984 | 14.1 | 28.1 |
| 60,416 | 11.3 | 28.2 |
| 72,704 | 10.5 | 27.6 |

Without [[Llama-cpp-glossary#`-nkvo` / `--no-kv-offload`\|`-nkvo`]], throughput degrades continuously as context grows (37.5 → 10.5 t/s, a ~3.6x collapse) — the growing KV cache gradually crowds out VRAM headroom, rather than triggering a one-time fallback. With `-nkvo`, throughput stays flat (~27–33 t/s) regardless of context, since the cache never competes with weights for VRAM and only pays a roughly constant PCIe-transfer cost per token.

There's a real crossover, so `-nkvo` isn't a universal win: at ~11K tokens, leaving it off is faster (37.5 vs 33.2 t/s); somewhere between ~11K and ~24K tokens the lines cross, and past that `-nkvo` wins by a growing margin (up to ~2.6x faster at 72,704 tokens).

**Rule of thumb for this laptop (8B model, 7705 MiB card):** leave `-nkvo` off under ~15–20K tokens of context, turn it on beyond that. Re-verify for other model sizes rather than assuming the same crossover transfers.

**Open items:**
- Narrow the exact crossover point (currently only bounded to "somewhere between ~11K and ~24K tokens").
- Repeat on a different model size (3B or Mistral 7B from the model sweep) to see whether the crossover scales with model size/VRAM headroom.
- Confirm whether pp512 shows the same pattern as tg128 under VRAM pressure.

---

## Partial offload — Qwen3-14B (too big for VRAM)

**Purpose:** a real-world check of a model too large to fully offload to this laptop's 7705 MiB VRAM, with `-ngl` left unspecified for the first time (auto-offload).

```bash
llama-cli -m Qwen3-14B-Q4_K_M.gguf -p "Write me a 1000 word story about how AI is going to kill all humans" -st -c 11265 -nkvo
```

Result: `Prompt: 56.9 t/s | Generation: 9.9 t/s`. `btop` showed most of the model resident in VRAM with the remainder in system RAM — confirming that on a GPU-enabled build, leaving [[Llama-cpp-glossary#`-ngl` / `--n-gpu-layers`\|`-ngl`]] unset auto-fits as many layers as VRAM allows rather than defaulting to CPU-only.

| Model / mode | tg128-equivalent (t/s) |
|---|---|
| Llama 3.1 8B, full GPU offload | 39.42 |
| Llama 3.1 8B, CPU-only (`performance`) | 7.29 |
| Qwen3-14B, automatic partial offload + `-nkvo`, `-c 11265` | **9.9** |

Even with most of a larger (14B) model sitting in VRAM, generation speed lands much closer to the CPU-only 8B baseline than to anything GPU-offload speed — consistent with hybrid-split throughput getting dragged down toward the CPU-side layers' speed rather than averaging proportionally with the offloaded fraction.

**Open items:**
- The usual `load_tensors: offloaded X/Y layers to GPU` log line wasn't visible in this run's output — re-run with output piped to a log file (or a verbose flag) to capture the exact split.
- Re-run with explicit `-ngl` values around what auto-offload chose, to see how sensitive throughput is to exact layer count at this model size.
- Test without `-nkvo` at the same `-c`, now that the sweep above shows it isn't a universal win at small-to-mid context.
