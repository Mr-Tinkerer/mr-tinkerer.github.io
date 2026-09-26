---
title: llama.cpp Glossary
description: Terms used across the Llama.cpp bucket.
tags:
  - glossary
---
GPU-offload and power-profile terms specific to running llama.cpp on the Gigabyte laptop live in [[Daily-driver-laptop/Llama.cpp/Llama-cpp-glossary|the laptop's own glossary]] instead of here, since they don't apply to this VM's CPU-only setup — link there rather than duplicating.

## llama.cpp

A CPU/GPU-capable LLM inference engine written in C/C++, exposing direct control over threading, batch size, quantization, and build-time CPU optimization flags. See [llama.cpp (GitHub)](https://github.com/ggml-org/llama.cpp). Also installed and running with GPU acceleration on the [[Daily-driver-laptop/Llama.cpp/Installation|Gigabyte laptop]] — a separate installation with its own numbers, not comparable to this VM's CPU-only figures.

## Ollama

A wrapper around llama.cpp adding a model manager, REST API, and simple CLI. Trades fine-grained performance tuning for convenience. See [Ollama (official site)](https://ollama.com/).

## GGUF

The file format llama.cpp (and Ollama, underneath) uses for model weights — one file bundles quantized weights, tokenizer, and metadata. See [GGUF format (Hugging Face docs)](https://huggingface.co/docs/hub/gguf).

## SIMD (Single Instruction, Multiple Data)

A category of CPU instruction that performs the same operation on multiple data points in a single instruction cycle. Matrix multiplication — the core operation in neural-network inference — benefits enormously from SIMD, which is why CPU inference speed depends heavily on which SIMD extensions the CPU (and the software build) support. AVX/AVX2 (below) are the specific x86 SIMD extensions that matter for this hardware. See [SIMD (Wikipedia)](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data).

## AVX / AVX2

x86 SIMD CPU instruction extensions. AVX2 (256-bit-wide operations, common from ~2013 onward) gives a large speedup for the matrix-multiplication-heavy workload of LLM inference over a CPU with no such extension. AVX-512 (512-bit-wide, roughly 2x AVX2's per-instruction throughput) is only present on newer/higher-end chips and isn't universal — always compile for the newest/widest extension the target CPU actually supports; compiling for more than it has will crash or silently fall back to a slower path. See [Advanced Vector Extensions (Wikipedia)](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions).

## x86-64 microarchitecture level

A standardized shorthand (v1–v4) for which x86 instruction-set extensions can be assumed present, used by compilers and hypervisor CPU-type settings — e.g. `x86-64-v3` guarantees AVX2, `x86-64-v4` guarantees AVX-512. Matters most in virtualized environments (see Virtual CPU type below), where the hypervisor — not the physical hardware — decides which flags the guest OS is allowed to see. See [x86-64 microarchitecture levels (Wikipedia)](https://en.wikipedia.org/wiki/X86-64#Microarchitecture_levels).

## Virtual CPU type (hypervisor)

When a VM's CPU type is set to a generic/portable profile (an explicit microarchitecture level, or a generic baseline model) instead of `host`/host-passthrough, the guest OS only sees the flags in that profile, even if the physical CPU supports more. Host passthrough maximizes performance but pins the VM to hardware with at least those exact flags (a live-migration concern); an explicit level (e.g. `x86-64-v3`) is a portable middle ground. Always verify what the guest actually sees (`cat /proc/cpuinfo | grep -o 'avx[0-9]*\|avx512[a-z]*' | sort -u`) rather than assuming from the configured type. See [[Homelab/Platform/Local-AI-VM/Fedora/Cpu-configuration|Fedora: CPU configuration]] for how this is set on the Local AI VM specifically.

## NUMA (Non-Uniform Memory Access)

Relevant only on multi-socket systems, where each physical CPU socket has its own directly-attached RAM and cross-socket access is slower. Not a factor on single-socket machines — which covers this VM's host, the Gigabyte laptop, and most of the other hardware in this vault — so related flags/settings can be ignored here. See [NUMA (Wikipedia)](https://en.wikipedia.org/wiki/Non-uniform_memory_access).

## KV cache

A memory buffer LLM inference engines allocate to store intermediate attention data, avoiding recomputation for every new token. Its size scales with the *configured maximum context size*, not the actual conversation length — an unset or very large context size can pre-allocate far more RAM than the model file's own size suggests. See [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks|Benchmarks]] for the practical RAM impact measured on this hardware, and [[Daily-driver-laptop/Llama.cpp/Benchmarks#`-nkvo` vs. context size — Llama 3.1 8B, `-ngl 999`|the Gigabyte laptop's benchmarks]] for the equivalent VRAM-pressure effect on a GPU build.

## mmap (memory-mapped file loading)

A lazy, on-demand file-loading strategy (used by llama.cpp by default) where a model file is mapped into the process's address space and only actually read from disk as each page is first touched, rather than fully read upfront. Disabling it (`--no-mmap`) forces a full upfront read, useful as a diagnostic step to isolate whether an issue is mmap-related. Tested directly on this VM and found *not* to be the dominant factor behind a slow NAS-hosted model run — see the Gotcha in [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks#Gotchas|Benchmarks]], where the real cause turned out to be an unset context size causing swap thrashing. See [mmap (Wikipedia)](https://en.wikipedia.org/wiki/Mmap).

## Context window

The maximum number of tokens a model can attend to in one inference pass — a shared budget across the system prompt, conversation history, current prompt, and generated output combined. Roughly 1 token ≈ 0.75 English words for typical prose, useful for translating a context-size number into a rough sense of how much text it represents.

## Quantization / GGUF quant naming (Q4_K_M, Q8_0, IQ3_M, etc.)

Compressing model weights from their native 16/32-bit floating point down to a lower bit-width (8-bit, 4-bit, etc.) to shrink RAM footprint and speed up memory-bandwidth-bound CPU inference, at some cost to output quality. In a GGUF filename, the leading number (Q3–Q8) is roughly the bits per weight; a `_K` suffix means "k-quants" (precision allocated non-uniformly, more bits to weights that matter more); a trailing `_S`/`_M`/`_L`/`XS` is a size/quality variant within that K-level. `IQ3_M` belongs to a newer "importance-matrix" quant family, generally squeezing better quality out of very low bit-widths than the older uniform `Q3` schemes. `Q4_K_M` is the community-standard default balance of size/speed/quality. See [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks#Quantization comparison (Llama 3.2 3B, `-t 4 -c 4096`)|Benchmarks]] for measured speed/quality numbers across quant levels on this hardware.

## Quantization-Aware Training (QAT) vs. Post-Training Quantization (PTQ)

Two ways a quantized model gets produced, sometimes signaled by a `-qat-` suffix in a GGUF release name. **PTQ** (the normal path, including most community GGUF conversions) trains at full precision first, then quantizes afterward as a separate step — the weights were never adjusted with quantization in mind. **QAT** simulates low-precision rounding *during* training itself, so the weights adapt to tolerate that precision loss from the start. A QAT model at a given bit-width typically retains noticeably more quality than a PTQ model at the same bit-width, but QAT can only be done by the original model creator as part of their own training pipeline.

## bitsandbytes / "bnb-4bit" — not a GGUF/llama.cpp format

A quantization library/format from the Hugging Face `transformers`/PyTorch ecosystem, unrelated to GGUF's own quant schemes. A `bnb-4bit` Hugging Face repo is for running/fine-tuning directly in Python via `transformers` + `bitsandbytes`, normally on GPU — it will not load in `llama-cli`/`llama-server`, which only read GGUF files. Look for a separate `-GGUF` repo of the same model instead. "unsloth" in a filename (e.g. `gemma-3-270m-unsloth-bnb-4bit`) just credits the uploader/toolchain, not the format itself — Unsloth also publishes genuine GGUF releases in many cases, so check the specific repo suffix, not the name.

## MLX — Apple's ML framework, not a GGUF/llama.cpp format

Apple's own ML framework, built for Apple Silicon (M-series Mac) hardware. Same category of mismatch as `bnb-4bit` above (wrong artifact type for llama.cpp), just targeting Apple Silicon instead of Nvidia/PyTorch GPUs. Look for the corresponding `-GGUF` repo of the same model.

## Unsloth Dynamic quantization ("UD" in a filename, e.g. `-UD-Q4_K_XL`)

Unsloth's own custom quantization method — selectively keeps sensitive layers/tensors at higher precision rather than a uniform bit-width, similar in goal to GGUF's own `_K` k-quants but a separate, Unsloth-branded pipeline. Still shipped as a normal `.gguf` file, loadable in llama.cpp like any other quant — unlike `bnb-4bit` or MLX above.

## Model naming conventions (size, date, context length)

Several unrelated pieces of information sometimes get baked directly into a model's filename — worth telling apart rather than assuming they all mean the same kind of thing:
- **Parameter count** ("7B", "3B") — see the Parameter count entry below.
- **"E" prefix / MatFormer effective-parameter naming** (e.g. "E2B", "E4B" in Gemma 3n) — stands for "Effective" parameters, not raw/total. An E2B model behaves like a 2B model in memory/compute footprint via selective parameter activation, but its true on-disk parameter count is larger (Gemma 3n E2B is ~5–6B raw). Budget RAM off the effective number; the file size on disk still reflects the larger true count.
- **Microsoft's size-tier naming** ("mini"/"small"/"medium" in Phi filenames) — relative tiers, not a cross-family standard unit. Phi-3's "mini" is specifically ~3.8B parameters (tracked elsewhere in this project as "Phi-3.5 Mini 3.8B") — translate to an actual parameter count before comparing across families.
- **Date stamps** (e.g. "0528" in `DeepSeek-R1-0528-Qwen3-8B-GGUF`) — an `MMDD` release date distinguishing an updated checkpoint from an earlier release of the "same" named model. Two repos sharing a base name but different date stamps aren't interchangeable — always check which dated checkpoint a benchmark or recommendation actually refers to.
- **Context length in a filename** (e.g. "4k"/"128k" in `Phi-3-mini-4k-instruct`) — some families release multiple context-window variants side by side, with the max context spelled out in the name, using the same K shorthand as the context-size table above. The longer-context variant usually trades a little efficiency/reliability within its normal range for the ability to handle much larger documents — pick the variant matching actual expected use, not always the largest. `-c` can still be set lower than a variant's max regardless of which one is downloaded.

## Google's "-it" naming suffix (e.g. `gemma-3-270m-it`)

Google's Gemma family's term for "instruction-tuned" — the same underlying meaning as the Base vs. Instruct model entry below, just family-specific naming (Llama uses "-Instruct", Qwen/Phi use "Instruct" as a separate word). Often combined with `-qat-` in the same release name; the two suffixes are independent (one is quantization method, the other is whether it's the instruction-following variant).

## Base model vs. "Instruct" model

Most model families release at least two variants of the same underlying weights. A **base model** is trained only to predict the next word — no inherent concept of "having a conversation," may ramble or continue a question with more questions rather than answering. An **Instruct model** (sometimes "Chat") is the base model further fine-tuned on conversational/instruction-following data so it reliably behaves like an assistant. Always use the Instruct variant for interactive use; base models are mainly for research or as a fine-tuning starting point.

## "Distill" in a model name (e.g. "DeepSeek-R1-Distill-Qwen")

Knowledge distillation: a smaller "student" model is trained to mimic a larger, more capable "teacher" model's behavior, transferring as much of the teacher's capability as possible into a cheaper-to-run package. Naming convention: the student base is named last, the teacher first — in "DeepSeek-R1-Distill-Qwen," Qwen is the student base and DeepSeek-R1 is the teacher whose reasoning behavior was distilled into it. A distilled model often inherits the teacher's specific demonstrated strength (e.g. chain-of-thought style) more strongly than it inherits broad general knowledge. Directly relevant on this hardware — see the Reasoning model entry and [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks#Accuracy benchmark — all 6 text models, 30-question test set|the accuracy benchmark]]'s DeepSeek-R1-Distill-Qwen 1.5B result.

## Abliterated model

A model that's had its built-in safety-refusal behavior surgically removed from the weights via a one-time edit (locating and projecting out an internal "refusal direction" in activation space), rather than through prompting or fine-tuning. Cheaper and faster than fine-tuning, which is part of why the technique spread quickly, but can cause collateral coherence/reasoning degradation since it's a fairly blunt edit — quality varies significantly by uploader/method, unlike the well-understood tradeoffs of quantization. Shows up as an `-abliterated` suffix, almost always from an independent uploader rather than the original model creator.

## Embedding model

A model that converts text into a fixed-length vector ("an embedding") positioned in vector space so semantically similar text ends up close together, rather than generating text itself. The engine behind the retrieval step in RAG (see below): documents are embedded once into a vector index, a query is embedded the same way, and the closest-matching vectors get pulled into context. Typically small and fast relative to chat models, making them cheap to run even alongside a larger local chat model on constrained hardware.

## Parameter count (e.g. "7B", "3B")

The number of weights in the model, in billions — the learned values used in the matrix multiplication that turns input tokens into output tokens. Every parameter must be stored in memory and computed against per generated token, which is why parameter count drives both RAM footprint and compute cost. Correlates with capability but isn't the whole story — training data quality and architecture/fine-tuning choices matter enormously too. On CPU (memory-bandwidth-bound, dense models), cost scales close to linearly with parameter count — confirmed directly on this VM (see [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks#Key findings — text models (all on this VM's CPU, thread-pinned)|Benchmarks]]: Llama 3.2 3B ran ~2.5x faster than Llama 3.1 8B, matching the ~2.5x parameter difference).

## Dense vs. Mixture of Experts (MoE) architecture

**Dense models** (e.g. Llama 3.2 3B, Llama 3.1 8B — everything benchmarked on this VM so far) compute every parameter for every token; compute cost scales directly with total size. **MoE models** replace large dense layers with multiple specialized "expert" sub-networks and a router that picks only 1–2 experts per token. An MoE model might have 17B total parameters but only 3B *active* per token — generation can feel as fast as a 3B model, but **100% of the weights must still reside in memory**, since MoE reduces compute per token, not RAM requirement.

## RAG (Retrieval-Augmented Generation) / "context stuffing"

Feeding relevant documents into a model's context window at inference time so it can reference them while answering, rather than relying only on training-time knowledge. Doesn't change the model's weights — the equivalent of handing someone a reference book right before asking a question. A small model with documents in context doesn't get generally "smarter," and the larger context needed to hold them means a larger KV cache (RAM cost, slower prompt processing) — on RAM-constrained hardware, stuffing in too many documents can reintroduce the same swap-thrashing problem as an oversized default context (see the Gotcha in [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks#Gotchas|Benchmarks]]). Also still bottlenecked by the model's own reading comprehension — accurate context doesn't guarantee a correct answer — and by the "lost in the middle" effect, where very long contexts cause under-weighting of details buried mid-document.

## Fine-tuning (contrast with RAG)

Actually updates a model's weights with additional training data, permanently baking in new knowledge/behavior, rather than supplying it at inference time. Far more RAM/compute-intensive than RAG and generally overkill for "answer questions about these documents" use cases, where RAG is the cheaper standard solution.

## Per-query retrieval vs. whole-context stuffing

Because a model is stateless between calls, a proper RAG pipeline does a retrieval step *before each question* — searching a document collection for just the chunks relevant to that specific question — rather than holding an entire document collection in context for a whole session. This keeps per-query context size (and RAM/KV-cache cost) small and roughly constant regardless of total collection size, at the cost of making retrieval quality the new bottleneck: if the wrong chunks are selected, the model will confidently answer based on what it was given, with no way to know what's missing. Ongoing conversation history is a separate concern from this and typically still needs to be retained for continuity even while reference documents are retrieved and dropped per-question.

## llama-bench

A benchmarking tool bundled with llama.cpp that reports measured tokens/sec (`pp512`, `tg128` — see below) for a given model/build/hardware/thread-count combination, plus standard deviation across repeated runs. Always trust a fresh `llama-bench` run on the actual target hardware over numbers quoted elsewhere.

## pp512 / tg128

`llama-bench`'s test names: `pp512` is **p**rompt **p**rocessing throughput on a 512-token synthetic prompt (the number after `pp`, configurable via `-p`, is the prompt length); `tg128` is **t**oken **g**eneration throughput producing 128 tokens (the number after `tg`, configurable via `-n`, is tokens generated). Both are reported in tokens/sec. See [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks|Benchmarks]] for the measured thread-scaling curves on this hardware, and the Prompt processing vs. token generation entry below for *why* the two scale so differently.

## Prompt processing vs. token generation

Two distinct phases of inference with different performance characteristics. **Prompt processing** (reading/encoding the input) is embarrassingly parallel and scales close to linearly with added threads/cores well past the point generation stops improving — confirmed directly on this VM (see the thread-scaling table in [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks#Thread-count scaling (same model, `llama-bench`, pinned threads 1–6)|Benchmarks]]). **Token generation** (producing output, one token at a time) is inherently sequential and generally memory-bandwidth-bound, so it hits diminishing returns from added threads much sooner. This is a common, expected pattern, not a bug — see Memory-bandwidth-bound vs. compute-bound below. Batch size (below) primarily tunes prompt-processing throughput.

## Batch size

How many tokens are processed together in a single forward pass (`-b`/`--batch-size` for the logical batch, `-ub`/`--ubatch-size` for the physical/micro-batch). Primarily affects prompt-processing throughput; has a smaller/different effect on generation speed. Worth tuning experimentally per machine rather than assuming a default is optimal.

## Memory-bandwidth-bound vs. compute-bound

CPU LLM inference — especially token generation — is typically limited by how fast weights can be streamed from RAM into CPU cache, not by how many floating-point operations the CPU can theoretically perform per second. This is why extra CPU threads stop helping generation speed well before they stop helping prompt processing, and why quantization helps so much: fewer bytes per weight means less data to move per token.

## Physical cores vs. logical threads (Hyperthreading/SMT)

A CPU with Hyperthreading/SMT reports more logical threads than physical cores (e.g. 6 physical cores → 12 logical threads). Compute-heavy matrix math scales mainly with physical cores; setting a thread count equal to the logical thread count usually doesn't produce a proportional speedup and can be slower due to contention. Start `-t` at the physical core count and benchmark ±1–2 from there.

## CPU pinning / affinity (Proxmox)

Dedicating specific physical host cores exclusively to one VM's cgroup, so it can never contend with (or starve) other VMs sharing the same host. On Proxmox this is done with `qm set <vmid> --affinity <range>` (applied live via cgroups/`taskset`) — not the `cpuset` key, which Proxmox silently accepts and writes into the VM's `.conf` file without actually enforcing it. Pinning is the strongest of a spectrum of isolation options: a softer alternative is capping a VM's *total* CPU time without dedicating specific cores, and the simplest option is just allocating the heavy VM fewer vCPUs than the host's physical core count, structurally guaranteeing some cores are never claimed by it. See [[Homelab/Platform/Local-AI-VM/Llama.cpp/Build-and-cpu-tuning#Pinning CPU cores|Build and CPU tuning]] for how this was diagnosed and fixed on the Local AI VM.

## Multimodal input / mmproj / mtmd

llama.cpp can accept images as input, but only for a vision-capable model (e.g. Gemma 3, Qwen2-VL, MiniCPM-V, LLaVA, InternVL) paired with a matching multimodal projector file (`mmproj-*.gguf`), loaded via `--mmproj` alongside `-m`. The projector is architecture-specific to its paired model and can't be mixed with a different model's weights. This tooling builds by default under `tools/mtmd` even in text-only builds; its presence doesn't mean it's in use. See [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks#Key findings — vision (multimodal) models|Benchmarks]] for measured results across vision models on this hardware.

## Non-text file input (Markdown, PDFs, other documents)

llama.cpp has no built-in file-ingestion pipeline beyond plain text tokens and image/audio via `mtmd` above — anything else must be converted to text first. Markdown/plain text/code files are trivial (`llama-cli -f file.md` just reads the raw contents as the prompt; Markdown syntax is seen as ordinary characters, not rendered). PDFs need either text extraction first (`pdftotext`, `pypdf` — cheap, loses layout/images) or treating pages as images through an OCR-capable multimodal model (preserves layout, adds full multimodal overhead). On RAM-constrained CPU-only hardware, text extraction is generally the far cheaper path.

## Office/document formats that don't reduce cleanly to text

Most office formats (`.pptx`, `.docx`, `.xlsx`, `.csv`) extract to text easily via mature libraries, since the underlying files are structured XML — visual layout is lost but text extraction is straightforward. Genuinely hard cases: slides/pages that are mostly diagrams/charts/screenshots with little native text, scanned/image-only documents with no text layer, and content where layout itself carries meaning (tables, flowcharts). Cheapest-to-most-expensive options: accept the lossy text-only extraction, OCR as a separate preprocessing step (`tesseract` — cheap, still loses layout), or render to images through a multimodal/OCR-capable vision model (preserves layout, full mmproj overhead per page).

## Reasoning / "thinking" model

A model trained to generate an extended chain-of-thought before its final answer, rather than answering directly — an axis layered on top of instruction-tuning, not a replacement for it, usually trained via reinforcement learning and/or distillation from a stronger reasoning teacher (see "Distill" above). The practical trade is "test-time compute": the model spends extra tokens/time thinking in exchange for better accuracy on hard problems, so its effective response time can be far higher than a similarly-sized non-reasoning model even at identical raw tokens/sec. Sharpens step-by-step logical deduction (math, logic puzzles) but doesn't reliably improve, and can worsen, tasks needing broad factual recall — a reasoning model can "confidently reason" its way to a fluent but fabricated answer when it lacks the underlying facts. See [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks#Key findings — text models (all on this VM's CPU, thread-pinned)|Benchmarks]] for this failure mode observed directly on this hardware (DeepSeek-R1-Distill-Qwen 1.5B).

## Shared CPU/GPU power budget on laptops

On many laptops — especially Nvidia Max-Q/Dynamic-Boost designs — the CPU and discrete GPU share a combined power/thermal budget rather than each having an independent ceiling, so an OS power profile tuned around sustained CPU performance can counterintuitively *reduce* GPU headroom (and vice versa) depending on which side of the budget a given workload stresses. Not applicable to this CPU-only VM (no discrete GPU), but directly measured and confirmed on the [[Daily-driver-laptop/Gigabyte-Gaming-A16-Ga6h/Power-profiles-and-gpu-performance|Gigabyte laptop]], including actual `nvidia-smi` power-limit numbers per profile.

## Build concepts

- **Compiling "natively" (`-march=native`)** — detects and targets the exact build machine's instruction set for the fastest possible binary, at the cost of portability to other CPUs. Fine for a single dedicated machine.
- **Precompiled/generic binaries** — prebuilt releases (e.g. Ollama's) target a broad compatibility baseline so they run everywhere out of the box, leaving some performance on the table versus a native build.
- **CMake configure vs. build** — `cmake -B <dir> [options]` inspects the system and generates build instructions (Makefiles) without compiling; `cmake --build <dir> [-j N]` actually compiles/links. `-j N` only controls build parallelism, not runtime speed.
- **Release vs. Debug build type** — Release enables optimizations and strips debug symbols (what you want for a performance-sensitive deployment); Debug disables optimizations and keeps debug info, useful only when actively debugging a crash.
- **Compiler warnings vs. errors** — a warning means the compiler flagged something but still produced a working binary; an error means it refused to produce one at all. Warnings in a large, fast-moving open-source project's own code are normal; only errors need fixing.
- **Incremental builds can silently use stale flags** — `make`-based builds decide whether to recompile based on file timestamps, not whether flags changed. Changing a significant build option (build type, `-march=native`, etc.) on an existing build directory can leave untouched object files linked in with the *old* flags, producing a binary whose cache/config looks correct but behaves like the old configuration. Fix: `rm -rf build` and rebuild fully after any such change, rather than incrementally. Directly caused the debug-flavored binary in [[Homelab/Platform/Local-AI-VM/Llama.cpp/Build-and-cpu-tuning#Incremental rebuild silently linked a debug-flavored binary|this VM's build]].
- **BLAS (Basic Linear Algebra Subprograms)** — a standardized interface for matrix/vector math, implemented by libraries like OpenBLAS or Intel MKL. llama.cpp's own hand-tuned `ggml` kernels (especially with a native build) are typically as fast or faster than an external BLAS library for CPU token generation; an external BLAS backend can sometimes help specifically with prompt processing on certain hardware. Not required for a first setup.
