---
title: llama.cpp Glossary
description: Terms used across the Llama.cpp bucket.
tags:
  - glossary
---
## llama.cpp

A CPU/GPU-capable LLM inference engine written in C/C++, exposing direct control over threading, batch size, quantization, and build-time CPU optimization flags. See [llama.cpp (GitHub)](https://github.com/ggml-org/llama.cpp).

## Ollama

A wrapper around llama.cpp adding a model manager, REST API, and simple CLI. Trades fine-grained performance tuning for convenience. See [Ollama (official site)](https://ollama.com/).

## GGUF

The file format llama.cpp (and Ollama, underneath) uses for model weights — one file bundles quantized weights, tokenizer, and metadata. See [GGUF format (Hugging Face docs)](https://huggingface.co/docs/hub/gguf).

## AVX / AVX2

x86 SIMD (Single Instruction, Multiple Data) CPU instruction extensions. AVX2 (256-bit-wide operations, common from ~2013 onward) gives a large speedup for the matrix-multiplication-heavy workload of LLM inference over a CPU with no such extension. See [Advanced Vector Extensions (Wikipedia)](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions).

## x86-64 microarchitecture level

A standardized shorthand (v1–v4) for which x86 instruction-set extensions can be assumed present, used by compilers and hypervisor CPU-type settings — e.g. `x86-64-v3` guarantees AVX2. See [x86-64 microarchitecture levels (Wikipedia)](https://en.wikipedia.org/wiki/X86-64#Microarchitecture_levels).

## KV cache

A memory buffer LLM inference engines allocate to store intermediate attention data, avoiding recomputation for every new token. Its size scales with the *configured maximum context size*, not the actual conversation length — an unset or very large context size can pre-allocate far more RAM than the model file's own size suggests. See [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks|Benchmarks]] for the practical RAM impact measured on this hardware.

## mmap (memory-mapped file loading)

A lazy, on-demand file-loading strategy (used by llama.cpp by default) where a model file is mapped into the process's address space and only actually read from disk as each page is first touched, rather than fully read upfront. Disabling it (`--no-mmap`) forces a full upfront read, useful as a diagnostic step to isolate whether an issue is mmap-related. See [mmap (Wikipedia)](https://en.wikipedia.org/wiki/Mmap).

## Context window

The maximum number of tokens a model can attend to in one inference pass — a shared budget across the system prompt, conversation history, current prompt, and generated output combined.

## Quantization / GGUF quant naming (Q4_K_M, Q8_0, IQ3_M, etc.)

Compressing model weights from their native 16/32-bit floating point down to a lower bit-width (8-bit, 4-bit, etc.) to shrink RAM footprint and speed up memory-bandwidth-bound CPU inference, at some cost to output quality. In a GGUF filename, the leading number (Q3–Q8) is roughly the bits per weight; a `_K` suffix means "k-quants" (precision allocated non-uniformly, more bits to weights that matter more); a trailing `_S`/`_M`/`_L`/`XS` is a size/quality variant within that K-level. `IQ3_M` belongs to a newer "importance-matrix" quant family, generally squeezing better quality out of very low bit-widths than the older uniform `Q3` schemes. `Q4_K_M` is the community-standard default balance of size/speed/quality. See [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks#Quantization comparison (Llama 3.2 3B, `-t 4 -c 4096`)|Benchmarks]] for measured speed/quality numbers across quant levels on this hardware.

## llama-bench

A benchmarking tool bundled with llama.cpp that reports measured tokens/sec (`pp512`, `tg128` — see below) for a given model/build/hardware/thread-count combination, plus standard deviation across repeated runs. Always trust a fresh `llama-bench` run on the actual target hardware over numbers quoted elsewhere.

## pp512 / tg128

`llama-bench`'s test names: `pp512` is **p**rompt **p**rocessing throughput on a 512-token synthetic prompt (the number after `pp`, configurable via `-p`, is the prompt length); `tg128` is **t**oken **g**eneration throughput producing 128 tokens (the number after `tg`, configurable via `-n`, is tokens generated). Both are reported in tokens/sec; prompt processing is embarrassingly parallel and scales with threads much further than generation, which is sequential and memory-bandwidth-bound. See [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks|Benchmarks]] for the measured thread-scaling curves on this hardware.

## Memory-bandwidth-bound vs. compute-bound

CPU LLM inference — especially token generation — is typically limited by how fast weights can be streamed from RAM into CPU cache, not by how many floating-point operations the CPU can theoretically perform per second. This is why extra CPU threads stop helping generation speed well before they stop helping prompt processing, and why quantization helps so much: fewer bytes per weight means less data to move per token.

## Physical cores vs. logical threads (Hyperthreading/SMT)

A CPU with Hyperthreading/SMT reports more logical threads than physical cores (e.g. 6 physical cores → 12 logical threads). Compute-heavy matrix math scales mainly with physical cores; setting a thread count equal to the logical thread count usually doesn't produce a proportional speedup and can be slower due to contention. Start `-t` at the physical core count and benchmark ±1–2 from there.

## CPU pinning / affinity (Proxmox)

Dedicating specific physical host cores exclusively to one VM's cgroup, so it can never contend with (or starve) other VMs sharing the same host. On Proxmox this is done with `qm set <vmid> --affinity <range>` (applied live via cgroups/`taskset`) — not the `cpuset` key, which Proxmox silently accepts and writes into the VM's `.conf` file without actually enforcing it. See [[Homelab/Platform/Local-AI-VM/Llama.cpp/Build-and-cpu-tuning#Pinning CPU cores|Build and CPU tuning]] for how this was diagnosed and fixed on the Local AI VM.

## Multimodal input / mmproj / mtmd

llama.cpp can accept images as input, but only for a vision-capable model (e.g. Gemma 3, Qwen2-VL, MiniCPM-V, LLaVA, InternVL) paired with a matching multimodal projector file (`mmproj-*.gguf`), loaded via `--mmproj` alongside `-m`. The projector is architecture-specific to its paired model and can't be mixed with a different model's weights. This tooling builds by default under `tools/mtmd` even in text-only builds; its presence doesn't mean it's in use. See [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks#Key findings — vision (multimodal) models|Benchmarks]] for measured results across vision models on this hardware.

## Reasoning / "thinking" model

A model trained to generate an extended chain-of-thought before its final answer, rather than answering directly — an axis layered on top of instruction-tuning, not a replacement for it. Sharpens step-by-step logical deduction (math, logic puzzles) but doesn't reliably improve, and can worsen, tasks needing broad factual recall — a reasoning model can "confidently reason" its way to a fluent but fabricated answer when it lacks the underlying facts. See [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks#Key findings — text models (all on this VM's CPU, thread-pinned)|Benchmarks]] for this failure mode observed directly on this hardware (DeepSeek-R1-Distill-Qwen 1.5B).
