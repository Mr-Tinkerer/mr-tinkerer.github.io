---
title: Llama.cpp Glossary
description: Core Llama.cpp terminology and concepts that apply across multiple Llama.cpp instances.
tags:
  - llama.cpp
  - glossary
  - Generic
---

# Overview

This glossary documents core Llama.cpp terminology and concepts that apply across multiple Llama.cpp instances (CPU and GPU-enabled systems). Hardware-specific tuning, benchmarks, and machine-specific configurations remain in their respective domains.

---

## llama.cpp

A CPU/GPU-capable LLM inference engine written in C/C++, exposing direct control over threading, batch size, quantization, and build-time CPU optimization flags. See [llama.cpp (GitHub)](https://github.com/ggml-org/llama.cpp).

---

## GGUF

The file format llama.cpp uses for model weights — one file bundles quantized weights, tokenizer, and metadata. See [GGUF format (Hugging Face docs)](https://huggingface.co/docs/hub/gguf).

---

## SIMD (Single Instruction, Multiple Data)

A category of CPU instruction that performs the same operation on multiple data points in a single instruction cycle. Matrix multiplication — the core operation in neural-network inference — benefits enormously from SIMD, which is why CPU inference speed depends heavily on which SIMD extensions the CPU supports. AVX/AVX2 are the specific x86 SIMD extensions that matter for typical hardware. See [SIMD (Wikipedia)](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data).

---

## AVX / AVX2

x86 SIMD CPU instruction extensions. AVX2 (256-bit-wide operations, common from ~2013 onward) gives a large speedup for the matrix-multiplication-heavy workload of LLM inference over a CPU with no such extension. AVX-512 (512-bit-wide, roughly 2x AVX2's per-instruction throughput) is only present on newer/higher-end chips and isn't universal — always compile for the newest/widest extension the target CPU actually supports. See [Advanced Vector Extensions (Wikipedia)](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions).

---

## x86-64 Microarchitecture Level

A standardized shorthand (v1–v4) for which x86 instruction-set extensions can be assumed present, used by compilers and hypervisor CPU-type settings — e.g. `x86-64-v3` guarantees AVX2, `x86-64-v4` guarantees AVX-512. Matters most in virtualized environments, where the hypervisor — not the physical hardware — decides which flags the guest OS is allowed to see. See [x86-64 microarchitecture levels (Wikipedia)](https://en.wikipedia.org/wiki/X86-64#Microarchitecture_levels).

---

## KV Cache

A memory buffer LLM inference engines allocate to store intermediate attention data, avoiding recomputation for every new token. Its size scales with the *configured maximum context size*, not the actual conversation length — an unset or very large context size can pre-allocate far more RAM than the model file's own size suggests. See the machine-specific benchmarks for practical RAM/VRAM impact.

---

## mmap (Memory-Mapped File Loading)

A lazy, on-demand file-loading strategy (used by llama.cpp by default) where a model file is mapped into the process's address space and only actually read from disk as each page is first touched, rather than fully read upfront. Disabling it (`--no-mmap`) forces a full upfront read, useful as a diagnostic step to isolate whether an issue is mmap-related.

---

## Context Window

The maximum number of tokens a model can attend to in one inference pass — a shared budget across the system prompt, conversation history, current prompt, and generated output combined. Roughly 1 token ≈ 0.75 English words for typical prose.

---

## Quantization / GGUF Quant Naming (Q4_K_M, Q8_0, IQ3_M, etc.)

Compressing model weights from their native 16/32-bit floating point down to a lower bit-width (8-bit, 4-bit, etc.) to shrink RAM footprint and speed up memory-bandwidth-bound inference, at some cost to output quality. In a GGUF filename, the leading number (Q3–Q8) is roughly the bits per weight; a `_K` suffix means "k-quants" (precision allocated non-uniformly, more bits to weights that matter more); a trailing `_S`/`_M`/`_L`/`XS` is a size/quality variant within that K-level. `Q4_K_M` is the community-standard default balance of size/speed/quality.

---

## Quantization-Aware Training (QAT) vs. Post-Training Quantization (PTQ)

Two ways a quantized model gets produced. **PTQ** (the normal path, including most community GGUF conversions) trains at full precision first, then quantizes afterward as a separate step. **QAT** simulates low-precision rounding *during* training itself, so the weights adapt to tolerate that precision loss from the start. A QAT model at a given bit-width typically retains noticeably more quality than a PTQ model at the same bit-width.

---

## Base Model vs. "Instruct" Model

Most model families release at least two variants of the same underlying weights. A **base model** is trained only to predict the next word — no inherent concept of "having a conversation." An **Instruct model** (sometimes "Chat") is the base model further fine-tuned on conversational/instruction-following data so it reliably behaves like an assistant. Always use the Instruct variant for interactive use.

---

## Reasoning / "Thinking" Model

A model trained to generate an extended chain-of-thought before its final answer, rather than answering directly — an axis layered on top of instruction-tuning. The practical trade is "test-time compute": the model spends extra tokens/time thinking in exchange for better accuracy on hard problems. Sharpens step-by-step logical deduction but doesn't reliably improve tasks needing broad factual recall.

---

## Embedding Model

A model that converts text into a fixed-length vector positioned in vector space so semantically similar text ends up close together, rather than generating text itself. Typically small and fast relative to chat models, making them cheap to run even alongside a larger local chat model.

---

## Parameter Count (e.g. "7B", "3B")

The number of weights in the model, in billions. Every parameter must be stored in memory and computed against per generated token, which is why parameter count drives both RAM footprint and compute cost. Correlates with capability but isn't the whole story — training data quality and architecture matter enormously too.

---

## Dense vs. Mixture of Experts (MoE) Architecture

**Dense models** compute every parameter for every token; compute cost scales directly with total size. **MoE models** replace large dense layers with multiple specialized "expert" sub-networks and a router that picks only 1–2 experts per token. An MoE model might have 17B total parameters but only 3B *active* per token — generation can feel as fast as a 3B model, but **100% of the weights must still reside in memory**, since MoE reduces compute per token, not RAM requirement.

---

## RAG (Retrieval-Augmented Generation)

Feeding relevant documents into a model's context window at inference time so it can reference them while answering, rather than relying only on training-time knowledge. Doesn't change the model's weights — the equivalent of handing someone a reference book right before asking a question. A small model with documents in context doesn't get generally "smarter."

---

## llama-bench

A benchmarking tool bundled with llama.cpp that reports measured tokens/sec (`pp512`, `tg128`) for a given model/build/hardware/thread-count combination. Always trust a fresh `llama-bench` run on the actual target hardware over numbers quoted elsewhere.

---

## pp512 / tg128

`llama-bench`'s test names: `pp512` is **p**rompt **p**rocessing throughput on a 512-token synthetic prompt; `tg128` is **t**oken **g**eneration throughput producing 128 tokens. Both are reported in tokens/sec. See machine-specific benchmark pages for measured numbers.

---

## Prompt Processing vs. Token Generation

Two distinct phases of inference with different performance characteristics. **Prompt processing** (reading/encoding the input) is embarrassingly parallel and scales close to linearly with added threads/cores. **Token generation** (producing output, one token at a time) is inherently sequential and generally memory-bandwidth-bound, so it hits diminishing returns from added threads much sooner.

---

## Batch Size

How many tokens are processed together in a single forward pass (`-b`/`--batch-size`). Primarily affects prompt-processing throughput; has a smaller/different effect on generation speed. Worth tuning experimentally per machine.

---

## Memory-Bandwidth-Bound vs. Compute-Bound

CPU LLM inference — especially token generation — is typically limited by how fast weights can be streamed from RAM into CPU cache, not by how many floating-point operations the CPU can theoretically perform per second. This is why extra CPU threads stop helping generation speed well before they stop helping prompt processing, and why quantization helps so much.

---

## Physical Cores vs. Logical Threads (Hyperthreading/SMT)

A CPU with Hyperthreading/SMT reports more logical threads than physical cores (e.g. 6 physical cores → 12 logical threads). Compute-heavy matrix math scales mainly with physical cores; setting a thread count equal to the logical thread count usually doesn't produce a proportional speedup. Start `-t` at the physical core count.

---

## Multimodal Input / mmproj / mtmd

llama.cpp can accept images as input, but only for a vision-capable model (e.g. Gemma 3, Qwen2-VL, MiniCPM-V, LLaVA, InternVL) paired with a matching multimodal projector file (`mmproj-*.gguf`), loaded via `--mmproj` alongside `-m`. The projector is architecture-specific to its paired model and can't be mixed with a different model's weights.

---

## Non-Text File Input (Markdown, PDFs, Other Documents)

llama.cpp has no built-in file-ingestion pipeline beyond plain text tokens and image/audio via `mtmd` — anything else must be converted to text first. Markdown/plain text/code files are trivial (`llama-cli -f file.md` just reads the raw contents). PDFs need either text extraction first (`pdftotext`, `pypdf`) or treating pages as images through an OCR-capable multimodal model. On RAM-constrained CPU-only hardware, text extraction is generally the far cheaper path.

---

## Reasoning Model Failure Mode

A reasoning model can "confidently reason" its way to a fluent but fabricated answer when it lacks the underlying facts. This is a more dangerous failure mode than simply "wrong," since fabricated commands read as fluent and plausible. Do not trust reasoning models for factual recall tasks.
