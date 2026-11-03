---
title: Llama.cpp Benchmarks on Local-AI VM (Fedora)
description: Text and vision model throughput, accuracy, and context-size limits measured on the Local AI VM's CPU-only hardware.
tags:
  - Homelab
  - Llama.cpp
---

## Overview

This page summarizes Llama.cpp benchmark results on the Local AI VM's CPU-only hardware. Full benchmark tables, per-image grading notes, and raw experiment logs are maintained in the infrastructure repository.

See the Glossary for hardware/inference concepts (SIMD, KV cache, mmap, etc.).

## Data Sources

Exhaustive benchmark data is maintained in the infrastructure repository:

- [Local_AI_Model_Reference_Guide.md](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/Local_AI_Model_Reference_Guide.md) — Exhaustive benchmark tables
- [vision_benchmark_results.md](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/vision_benchmark_results.md) — Vision model results
- [ctx-oom-sweep-by-model.md](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/ctx-oom-sweep-by-model.md) — Context size OOM sweeps
- [experiment-log.md](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/experiment-log.md) — Experiment log
- [model-accuracy-test-set.md](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/model-accuracy-test-set.md) — Accuracy test set
- [benchmark-results-log.md](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Notes/benchmark-results-log.md) — Benchmark results log
- [Benchmark Scripts](https://github.com/Mr-Tinkerer/Project-Dump/tree/main/Homelab/Local-AI%20VM/Notes/Benchmark%20Scripts) — Benchmark scripts

For GPU-accelerated comparison, the Gigabyte laptop's benchmarks sweep the same model set on an RTX 5060 Laptop GPU. Hardware setups are not directly comparable (different RAM/VRAM ceilings, different build provenance), but the model list overlaps intentionally for like-for-like reference.

## Benchmark Methodology

### Model-Benchmark.sh

Downloads a fixed list of baseline models (one Q4_K_M quant each) and runs `llama-bench -t <threads>` against every one, capturing the standard `pp512`/`tg128` throughput pair per model with no other variables changed. This is the "one number per model" baseline sweep.

### Quantization-Benchmark.sh

Holds the model constant (Llama 3.2 3B) and sweeps quantization level (`Q8_0`, `Q4_K_S`, `IQ3_M`), running `llama-bench` for speed at each quant, then re-running 3 fixed prompts (logic riddle, strict-JSON generation task, coding task) through `llama-cli` at low temperature (`--temp 0.2`) for qualitative output degradation assessment.

### Accuracy-Benchmark.sh

Downloads all 6 baseline models, then runs the same 30-question set through every model at identical settings (`-c 4096`, `--temp 0.8`, same thread count), logging each raw response for hand-grading. The 30 questions span 6 categories: general knowledge/instruction-following, logic & math, coding, summarization/extraction, open-ended writing, and Linux sysadmin (beginner to advanced). The sysadmin category tests whether models fabricate plausible but wrong commands rather than hedge.

### ctx_sweep.sh

For each model, doubles `-c` starting at 512 across both `--no-mmap` and default-mmap modes independently, running each combination under `/usr/bin/time -v` while a background poller samples system-wide swap usage every 0.2s. Stops a given model/mode combination as soon as either the OS OOM-kills the process (exit code 137/kernel log "out of memory"/"oom-kill") or the process fails to start cleanly (non-zero, non-OOM exit). Both are treated as the memory ceiling for that combination.

### Image-Benchmark.sh

Downloads each vision model's GGUF and matching `mmproj` projector file, then runs a single fixed prompt ("Describe what you see in this image in detail.") against 15 fixed test images per model at `-c 4096`, timing wall-clock seconds per image and logging raw descriptions for hand-grading. The 15 images (kept locally, not migrated per image-curation rules) span photos, UI/game screenshots, structured technical CLI output, 3D-rendered character art, and handwritten text.

### Grading

All grading is by hand against a rubric (correctness/quality/format-compliance), not automated. Models that hedge ("I'm not sure") are treated as more trustworthy long-term than those that fluently fabricate.

## Key Findings — Text Models (CPU, Thread-Pinned)

### Model Size vs. Speed

Llama 3.2 3B ran ~2.5x faster than Llama 3.1 8B on both prompt processing and generation — roughly matching the ~2.5x parameter-count difference.

### Thread Scaling

Throughput scales with threads but not linearly. Going from 1 to 6 threads pinned, Llama 3.1 8B's generation speed roughly doubled (2.69 → 6.09 t/s) while prompt-processing speed scaled better (16.32 → 29.04 t/s). The 3B model showed a similar pattern.

### Accuracy vs. Speed Tradeoff

The largest model tested (Llama 3.1 8B) scored highest on the accuracy test set. Each smaller/faster model trades some accuracy for speed, down to DeepSeek-R1-Distill-Qwen 1.5B as the fastest and least accurate of the six.

### RAM Usage Ceiling

Actual RAM usage while a model is loaded and running can exceed the model file's own size once the KV cache and inference overhead are counted. On this 8GB VM, running near 99% RAM usage occurred with only a mid-size model loaded.

See [RAM usage screenshot](Images/local-ai-vm-ram-usage-near-oom-btop.png) for `btop` showing 7.68 GiB of 7.73 GiB RAM in use (99%) on the Local AI VM with `llama-cli` running a single model.

## Measured Throughput per Model (Q4_K_M, `-t 4`)

| Model | Params | pp512 (t/s) | tg128 (t/s) | Max safe `-c` on this 8GB VM |
|-------|--------|-------------|-------------|------------------------------|
| DeepSeek-R1-Distill-Qwen 1.5B | 1.5B | 119.80 | 26.29 | 262,144 (both mmap modes) |
| Gemma 2 2B | 2B | 75.77 | 14.99 | 131,072 (both mmap modes) |
| Llama 3.2 3B | 3B | 56.80–72.62 | 13.12–13.76 (varies by thread count) | 65,536 (both mmap modes) |
| Qwen 2.5 3B | 3B | 56.64 | 13.83 | 262,144 (both mmap modes) |
| Phi-3.5 Mini 3.8B | 3.8B | 34.75 | 11.38 | 16,384 with `--no-mmap`; 32,768 with default mmap |
| Llama 3.1 8B | 8B | 23.08–29.04 | 5.80–6.09 | 32,768 with `--no-mmap`; 131,072 with default mmap |
| Mistral 7B | ~7B | 22.45 | 6.18 | 16,384 with `--no-mmap`; 32,768 with default mmap |

For the same models under full GPU offload instead of CPU-only, see the Gigabyte laptop's model sweep — pp512/tg128 numbers there are one to two orders of magnitude higher, as expected for GPU vs. CPU inference.

## Thread-Count Scaling (Same Model, Pinned Threads 1–6)

| Threads | Llama 3.2 3B tg128 | Llama 3.2 3B pp512 | Llama 3.1 8B tg128 | Llama 3.1 8B pp512 |
|---------|-------------------|-------------------|-------------------|-------------------|
| 1 | 6.27 | 15.77 | 2.69 | 6.32 |
| 2 | 10.79 | 30.62 | 4.65 | 12.54 |
| 3 | 12.48 | 45.50 | 5.45 | 18.58 |
| 4 | 13.12 | 56.80 | 5.80 | 23.08 |
| 5 | 13.54 | 64.86 | 5.99 | 26.30 |
| 6 | 13.75 | 72.43 | 6.09 | 29.04 |

Both models show the same shape: prompt processing (`pp512`) scales almost linearly with threads all the way to 6 (embarrassingly parallel), while generation (`tg128`) shows strong diminishing returns past ~3 threads (a 6.09/6.09 ≈ +12% gain from 3→6 threads on the 8B model, doubling the thread count) — the memory-bandwidth-bound bottleneck saturates well before the compute does. This is why the Build-and-CPU-tuning page pins to 4 threads rather than the full core count: it costs ~5% generation speed while freeing 2 physical cores for other VMs. See the Build and CPU tuning page for the pinning decision.

## Quantization Comparison (Llama 3.2 3B, `-t 4 -c 4096`)

`Q8_0`, `Q4_K_S`, and `IQ3_M` quants of the same model were compared for both speed (`llama-bench`) and qualitative output (3 fixed prompts — logic riddle, strict-JSON generation task, coding task — at `--temp 0.2`). As expected, lower bit-width quants are smaller and faster to run but trade output fidelity, most noticeable on strict-format (JSON) and precise-logic tasks rather than open-ended writing. `Q4_K_M` (the default used everywhere else on this VM) sits between `Q8_0` and `Q4_K_S`/`IQ3_M` in the size/quality trade space.

## Accuracy Benchmark — All 6 Text Models, 30-Question Test Set

| Model | Overall Avg (/5) | General Knowledge | Logic & Math | Coding | Summarization | Writing | Linux Sysadmin |
|-------|-----------------|-------------------|--------------|--------|---------------|---------|----------------|
| Llama 3.1 8B | 4.83 | 4.62 | 5.00 | 4.88 | 4.88 | 4.83 | 4.82 |
| Phi-3.5 Mini 3.8B | 4.73 | 3.88 | 5.00 | 5.00 | 4.75 | 5.00 | 4.77 |
| Llama 3.2 3B | 4.50 | 4.50 | 5.00 | 4.00 | 4.12 | 4.83 | 4.55 |
| Qwen 2.5 3B | 4.45 | 4.25 | 5.00 | 5.00 | 4.38 | 5.00 | 4.00 |
| Gemma 2 2B | 4.03 | 4.00 | 5.00 | 4.06 | 4.00 | 4.17 | 3.64 |
| DeepSeek-R1-Distill-Qwen 1.5B | 2.18 | 3.12 | 3.12 | 3.62 | 2.88 | 2.83 | 0.55 |

### Findings

- Google Gemini research helped identify which model families were worth downloading and benchmarking rather than guessing blind from model names alone
- **Llama 3.1 8B and Phi-3.5 Mini 3.8B were the strongest all-rounders**, both scoring highly in every category. Given the 8B model's 6.09 tg128 t/s ("comfortable reading pace" @ 6 threads), it's the best default for general assistant/sysadmin use on this hardware, with Phi-3.5 Mini as a faster (11.38 tg128 t/s), nearly-as-accurate fallback
- **DeepSeek-R1-Distill-Qwen 1.5B shows split personality:** strong on logic/math (clean chain-of-thought, correct answers) but catastrophic on Linux sysadmin questions (0.55/5) — confidently fabricating nonexistent commands and flags on nearly every sysadmin question rather than hedging. This is a more dangerous failure mode than simply "wrong," since fabricated commands read as fluent and plausible. **Do not trust this model for sysadmin/Linux command questions**, despite its excellent raw speed (26.29 tg128 t/s) and math ability — a direct illustration of reasoning training sharpening deduction without improving factual recall
- **Two concrete technical errors surfaced in otherwise-strong models:** Qwen 2.5 3B dropped the minute field in a cron schedule answer (would run every minute during the 2am hour instead of once daily), and separately recommended flushing all firewall rules just to add one port rule
- **Phi-3.5 Mini had one isolated instruction-following miss** — asked to explain weather vs. climate in exactly two sentences, it answered correctly then spiraled into a large fabricated wall of unrequested follow-up content
- **Grading caveat:** Grading is by hand against a rubric, not automated. Some "most moons" answers cite Jupiter (~79–92 moons), correct as of these models' training cutoffs, though Saturn overtook Jupiter's count after a large 2023 discovery batch — not counted as a model error since it reflects training-data recency, not a reasoning failure

## Key Findings — Vision (Multimodal) Models

Fifteen fixed test images (photos, screenshots, renders, handwritten text) were run through several vision-capable models and hand-graded 1–5 for description accuracy:

- **Qwen2-VL-7B** was the most accurate (4.85/5) but by far the slowest (~206s/image)
- **MiniCPM-V-2** had the best accuracy-per-parameter, beating larger models at under 3B parameters
- **Gemma 3 4B** was the best speed/accuracy balance among models that reliably completed
- **LLaVA-1.6 13B** could not load at all (OOM-killed) on this VM's 8GB RAM budget
- **InternVL2 8B+** loaded but its responses were truncated on every test image, making it ungraded rather than low-scoring
- Larger/higher-resolution images slowed every model down substantially (5–10x on the two largest test images), sometimes causing truncated responses even in otherwise-accurate models

## Key Findings — Safe Context Size (`-c`) per Model

Maximum safe context size before hitting an out-of-memory condition varies enormously by model — from 16,384 tokens (Phi-3.5 Mini and Mistral with `--no-mmap`) up to 524,288 tokens (Gemma 3 4B) on this same 8GB VM, driven by the KV cache size scaling with configured context rather than model size alone. Two mostly-consistent patterns emerged:

- **Bigger base models OOM at smaller context sizes** — Mistral-7B and Llama 3.1 8B hit their ceilings in the tens of thousands of tokens, while much smaller models (DeepSeek-R1-Distill-Qwen 1.5B, Qwen 2.5 3B, Gemma 3 4B) tolerated context sizes two orders of magnitude larger before OOMing, because a bigger base model leaves less RAM headroom for the KV cache before the combined footprint exceeds 8GB
- **Default mmap mode consistently tolerated a somewhat larger context than `--no-mmap`** before OOMing (e.g. Mistral-7B: 32,768 vs. 65,536) — consistent with mmap's lazy paging letting cold model-weight pages be dropped/reloaded under memory pressure rather than being pinned in RAM the way a fully `--no-mmap`-loaded copy is. Treated as a plausible mechanism, not a fully confirmed one

See the linked `ctx-oom-sweep-by-model.md` for the full per-model table. The Gigabyte laptop shows an analogous but distinct ceiling: rather than degrading gracefully via swap like this VM's system RAM, a VRAM-based KV cache on that GPU either fails outright at load time when oversized, or (per the `-nkvo` sweep) drags throughput down continuously as it crowds out VRAM.

## Gotchas

### Unset Context Size Symptom

An unset `-c` (context size) can look like a NAS/network or mmap bottleneck when it's actually RAM pressure. A real-world `llama-cli` run against a model file on a NAS share showed prompt processing ~9x slower than pure `llama-bench` numbers and unexpectedly low CPU utilization (133%, barely over 1 core) — pointing first toward a network-mmap round-trip theory. Moving the model to local disk barely changed anything. The actual cause: no `-c` was set, so llama.cpp defaulted to the model's large native max context, pre-allocating a KV cache far bigger than the short exchange needed and pushing the 8GB VM into swap thrashing (millions of major page faults, RAM pinned at 98–99%). Setting `-c 4096` explicitly dropped max RSS from ~7.2–7.6GB to ~2.4GB, eliminated swapping, and restored expected CPU utilization and throughput.

**Always set `-c` explicitly to the actual expected use case rather than leaving it at a model's default maximum**, especially on RAM-constrained hardware — and when real-world performance looks anomalous, check RAM/swap activity (`btop`/`free -h`) before chasing an I/O or network explanation, since swap thrashing produces symptoms (low CPU%, high major page faults) that superficially resemble an I/O bottleneck elsewhere.

### Kernel Log Contamination

A batch of context-sweep results had to be discarded and re-run because they shared duplicated/reused kernel-log evidence across different models (a logging artifact of not clearing `dmesg`/journal between runs) rather than genuine independent measurements. Affected: Meta-Llama-3.1-8B (`--no-mmap`), Phi-3.5-mini (both modes), Qwen2.5-3B (`--no-mmap`), Gemma-3-4B (`--no-mmap`), Llama-3.2-3B (both modes), gemma-2-2b (both modes) — all were re-run with a fresh `dmesg -c` (clear-on-read) immediately after each individual run before being trusted.

### Open/Unresolved

Meta-Llama-3.1-8B's `mmap (default)` ceiling shows a contradiction between sweep runs at `-c 65536` (one run OOM-killed, another completed cleanly) that hasn't been re-tested a third time to resolve — treat its 131,072 mmap ceiling as provisional until confirmed.
