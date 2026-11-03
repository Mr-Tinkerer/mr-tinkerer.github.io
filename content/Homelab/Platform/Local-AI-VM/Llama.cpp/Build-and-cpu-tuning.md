---
title: Llama.cpp Build and CPU Tuning on Local-AI VM (Fedora)
description: Building llama.cpp from source on the Local AI VM and tuning it for CPU-only inference with no GPU.
tags:
  - Homelab
  - Llama.cpp
---

## Overview

This guide covers building llama.cpp from source on the Local AI VM for CPU-only inference, including build configuration, installation, and CPU tuning.

See the Ollama glossary entry for the wrapper alternative. The Gigabyte laptop runs a separate llama.cpp install with GPU acceleration — its build path and tuning knobs (`-ngl`, `-nkvo`, power profiles) are different enough to live in their own bucket.

See the Fedora CPU configuration page for how this VM's virtual CPU type and core pinning are set at the Proxmox/guest-OS level — those facts apply regardless of what's running on the VM.

## Why llama.cpp Instead of Ollama

The Local AI VM has no GPU passthrough, so getting the most out of CPU-only inference means using llama.cpp directly rather than Ollama's wrapper, for direct control over threading, batch size, and the CPU instruction set it's compiled for.

## Building

### Dependencies

```bash
sudo dnf install -y gcc gcc-c++ cmake git make libcurl-devel
```

- **gcc/gcc-c++** — Required C/C++ compilers
- **cmake** — llama.cpp's build system generator
- **git** — To clone/update the repo
- **make** — CMake's default generator on Linux
- **libcurl-devel** — Only needed for llama.cpp's `--hf-repo`-style direct downloads; not required when sourcing models from elsewhere (e.g. NAS), but cheap to install upfront to avoid a rebuild later

### Build Configuration

llama.cpp was built with:

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release -DGGML_NATIVE=ON
cmake --build build --config Release -j <physical core count>
```

### Build Options

- **`-DGGML_NATIVE=ON`** — Auto-detects and targets the exact build machine's instruction set (AVX2, FMA, SSE4.2, etc.) — the mechanism is CMake passing `-march=native` through to GCC. This is equivalent in spirit to `-march=native` and is only appropriate for a binary that will only ever run on this exact machine (a portable binary for different hardware would instead target an explicit level, e.g. AVX2-only flags, not NATIVE)
- **`-j <N>`** — Only controls build parallelism (matched to physical core count to keep the build itself fast) — has no effect on the resulting binary's runtime performance
- **Multimodal tooling** — The build also compiles a large set of multimodal/vision-model tooling (`tools/mtmd`) by default, even when only text inference is needed

### Verifying AVX2 Compilation

`llama-cli --version` only prints build/compiler info, not CPU feature detection. Checking `build/CMakeCache.txt` for `GGML_AVX2` can be misleading too: with `GGML_NATIVE=ON`, those manual feature-selection flags are bypassed entirely and show `OFF` even though native detection is working. The real confirmation is in the actual compile flags:

```bash
grep -i "march\|mavx" build/CMakeFiles/ggml-cpu.dir/flags.make
```

Which should show `-march=native` in both `CXX_FLAGS` and `C_FLAGS` — on a CPU with AVX2 (present since Haswell, 2013), `-march=native` always resolves to include it. The `system_info:` line printed at runtime when a model loads (`AVX2 = 1`) is the final, undeniable confirmation.

This VM's build was self-compiled by design, for exactly this kind of flag-level control. The Gigabyte laptop, by contrast, uses prebuilt CachyOS/Arch packages — a deliberately different tradeoff (convenience over guaranteed build provenance).

## Installing to the Path

The built source tree was moved to `/opt` (conventional location for manually-installed, non-package-manager software), with ownership handed back to the regular build user (`chown -R`, since `sudo mv` leaves root as owner), and its binaries (`llama-server`, `llama-cli`, `llama-bench`) symlinked into `/usr/local/bin`. Symlinking rather than copying means future in-place rebuilds are picked up automatically.

### Shared Library Error

This surfaced a missing shared-library error at runtime:

```
llama-server: error while loading shared libraries: libllama-server-impl.so: cannot open shared object file: No such file or directory
```

llama.cpp builds its core logic as shared libraries (`libllama-server-impl.so`, `libggml.so`, `libllama.so`, etc.) that live in `build/bin/` alongside the executables rather than being statically linked in. Symlinking just the binary doesn't help, because the dynamic linker resolves shared-library dependencies against a fixed set of system paths (plus `LD_LIBRARY_PATH`/the `ldconfig` cache) — `/opt/llama.cpp/build/bin` isn't one of them by default.

### Resolution: ldconfig

Register the build directory with `ldconfig` (system-wide, permanent):

```bash
echo "/opt/llama.cpp/build/bin" | sudo tee /etc/ld.so.conf.d/llama-cpp.conf
sudo ldconfig
```

Verify with `llama-server -v` and `ldconfig -p | grep llama`. If `build/` is ever wiped and reconfigured/rebuilt, the `.so` files regenerate at the same path, so this entry keeps working — re-run `sudo ldconfig` after any rebuild as a safety habit.

### Alternatives Considered

- **LD_LIBRARY_PATH env var** — Session/service-scoped only, easy to forget in a new shell or future systemd unit
- **Symlinking each .so individually into /usr/local/lib** — Same ongoing maintenance burden as the ldconfig fix, no real advantage

## CPU Core Pinning

The benchmark and serving commands pin llama.cpp to a fixed number of CPU threads (`-t`) rather than letting it use every core, to get consistent, comparable benchmark numbers and leave headroom for the rest of the VM. See the Benchmarks page for the thread-scaling data this choice was based on (generation speed saturates around 3–4 threads on this hardware; prompt processing keeps scaling further). See the Fedora CPU pinning and core isolation page for the VM-level vCPU/`affinity` pinning this thread count was chosen alongside.

## Gotchas

### Incremental Rebuild Silently Linked a Debug-Flavored Binary

The very first `llama-bench` run reported console warnings (`asserts enabled`, `debug build`) despite `CMakeCache.txt` correctly showing `CMAKE_BUILD_TYPE=Release` — numbers from that run were roughly 8x slower than the real baseline and had to be discarded.

#### Root Cause

CMake's default Linux generator (`make`) decides whether to recompile a source file based on file timestamps, not on whether compiler flags changed. An earlier configure (before `-DCMAKE_BUILD_TYPE=Release` was set) had already compiled some object files with debug/assert flags; re-running `cmake -B build` with `Release` correctly updated the cache and `flags.make`, but `make` didn't consider those already-compiled `.o` files stale, so they were linked as-is into the final binary alongside newly-recompiled Release object files — a binary that reports `Release` in its build cache while still containing debug-compiled code.

#### Fix

After changing a significant CMake option (build type, native flags, etc.) on an existing build directory, do a full `rm -rf build` + reconfigure + rebuild rather than an incremental one, to guarantee every object file is compiled under the current, correct flags.

**Not the same cause everywhere:** The same `asserts enabled, performance may be affected` warning also shows up on the Gigabyte laptop, but for a different, unrelated reason — that machine uses a prebuilt Arch package whose `llama-cpp` PKGBUILD explicitly sets `-DCMAKE_BUILD_TYPE=None` (a deliberate upstream packaging choice, confirmed by reading the PKGBUILD directly), not a stale incremental build like this VM's.

### Distro-Packaged llama-bench/llama-cli is the Wrong Build for This Hardware

A precompiled `llama-bench` available via the distro's own package repo (separate from the self-built `./build/bin/llama-bench`) was tried as a comparison point. It reported `backend: ROCm` (AMD's GPU compute framework) and printed `ggml_cuda_init: failed to initialize ROCm: no ROCm-capable device is detected`, then produced drastically worse numbers (pp512: 5.12 t/s, tg128: 4.39 t/s) than the self-built CPU binary (~55-56 t/s pp512 / ~13 t/s tg128 at the same thread count). Passing `-ngl 0` (force zero GPU layers) did not fix it — the binary still attempted the ROCm backend regardless. This machine has no AMD GPU at all, so the distro package is fundamentally the wrong build variant here, independent of any flags — not pursued further.

**All benchmark numbers for this project use the self-built binary.**

The Gigabyte laptop's distro-packaged install is a different situation: that machine actually has the matching NVIDIA/CUDA hardware the package targets, so the same "wrong backend for this hardware" problem doesn't apply there.

### Compiler Warnings During the Build Are Expected and Safe to Ignore

The build produces warnings such as `-Wdeprecated-enum-enum-conversion` and `-Wdeprecated-declarations` from llama.cpp's own upstream code (a bitwise OR between two different enum types; use of a deprecated filesystem API) — normal in a fast-moving open source project, and not something to chase down as long as the build completes and links (`Built target ...` lines print, no compiler errors). Only errors (the build stops, no binary produced) need fixing.

---

See the Benchmarks page for the resulting throughput numbers across models and thread counts, and the Serving as a service page for running it persistently. For the GPU-accelerated build on different hardware, see the Gigabyte laptop's Llama.cpp bucket.
