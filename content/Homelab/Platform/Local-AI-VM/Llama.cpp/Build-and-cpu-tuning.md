---
title: Build and CPU tuning
description: Building llama.cpp from source on the Local AI VM and tuning it for CPU-only inference with no GPU.
tags:
  - Homelab
  - Llama.cpp
---
# Why llama.cpp instead of Ollama

The [[Homelab/Logical/Virtual-machines/Vm-allocation|Local AI VM]] has no GPU passthrough, so getting the most out of CPU-only inference meant using [[Homelab/Platform/Local-AI-VM/Llama.cpp/Llama-cpp-glossary#llama.cpp|llama.cpp]] directly rather than [[Homelab/Platform/Local-AI-VM/Llama.cpp/Llama-cpp-glossary#Ollama|Ollama]]'s wrapper, for direct control over threading, batch size, and the CPU instruction set it's compiled for. The [[Daily-driver-laptop/Llama.cpp/Installation|Gigabyte laptop]] runs a separate llama.cpp install with GPU acceleration — its build path and tuning knobs (`-ngl`, `-nkvo`, power profiles) are different enough to live in their own bucket rather than here; see that page for the GPU-specific side of things.

See [[Homelab/Platform/Local-AI-VM/Fedora/Cpu-configuration|Fedora: CPU configuration]] for how this VM's virtual CPU type and core pinning are set at the Proxmox/guest-OS level — those facts apply regardless of what's running on the VM, so I keep them with the Fedora guest OS bucket rather than here.

# Building

```bash
sudo dnf install -y gcc gcc-c++ cmake git make libcurl-devel
```

- `gcc`/`gcc-c++` — required C/C++ compilers.
- `cmake` — llama.cpp's build system generator.
- `git` — to clone/update the repo.
- `make` — CMake's default generator on Linux.
- `libcurl-devel` — only needed for llama.cpp's own `--hf-repo`-style direct downloads; not required when sourcing models from elsewhere (e.g. NAS), but cheap to install upfront to avoid a rebuild later.

llama.cpp was built with:

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release -DGGML_NATIVE=ON
cmake --build build --config Release -j <physical core count>
```

- `-DGGML_NATIVE=ON` auto-detects and targets the exact build machine's instruction set (AVX2, FMA, SSE4.2, etc.) — the mechanism is CMake passing `-march=native` through to GCC. This is equivalent in spirit to `-march=native` and is only appropriate for a binary that will only ever run on this exact machine (a portable binary for different hardware would instead target an explicit level, e.g. AVX2-only flags, not `NATIVE`).
- `-j <N>` only controls build parallelism (matched to physical core count to keep the build itself fast) — it has no effect on the resulting binary's runtime performance.
- The build also compiles a large set of multimodal/vision-model tooling (`tools/mtmd`) by default, even when only text inference is needed.

**Verifying AVX2 actually got compiled in** — `llama-cli --version` only prints build/compiler info, not CPU feature detection. Checking `build/CMakeCache.txt` for `GGML_AVX2` can be misleading too: with `GGML_NATIVE=ON`, those manual feature-selection flags are bypassed entirely and show `OFF` even though native detection is working. The real confirmation is in the actual compile flags:

```bash
grep -i "march\|mavx" build/CMakeFiles/ggml-cpu.dir/flags.make
```

which should show `-march=native` in both `CXX_FLAGS` and `C_FLAGS` — on a CPU with AVX2 (present since Haswell, 2013), `-march=native` always resolves to include it. The `system_info:` line printed at runtime when a model loads (`AVX2 = 1`) is the final, undeniable confirmation.

This VM's build was self-compiled by design, for exactly this kind of flag-level control. The Gigabyte laptop, by contrast, uses [[Daily-driver-laptop/Llama.cpp/Installation|prebuilt CachyOS/Arch packages]] instead — a deliberately different tradeoff (convenience over guaranteed build provenance), which is also why that machine's "asserts enabled" warning (below) traces to a different cause than this VM's: an upstream packaging choice rather than a stale incremental build.

# Installing to the path

The built source tree was moved to `/opt` (conventional location for manually-installed, non-package-manager software), with ownership handed back to the regular build user (`chown -R`, since `sudo mv` leaves root as owner), and its binaries (`llama-server`, `llama-cli`, `llama-bench`) symlinked into `/usr/local/bin` — symlinking rather than copying means future in-place rebuilds are picked up automatically.

This surfaced a missing shared-library error at runtime:

```
llama-server: error while loading shared libraries: libllama-server-impl.so: cannot open shared object file: No such file or directory
```

llama.cpp builds its core logic as shared libraries (`libllama-server-impl.so`, `libggml.so`, `libllama.so`, etc.) that live in `build/bin/` alongside the executables rather than being statically linked in. Symlinking just the binary doesn't help, because the dynamic linker resolves shared-library dependencies against a fixed set of system paths (plus `LD_LIBRARY_PATH`/the `ldconfig` cache) — `/opt/llama.cpp/build/bin` isn't one of them by default.

**Fix used:** register the build directory with `ldconfig` (system-wide, permanent):

```bash
echo "/opt/llama.cpp/build/bin" | sudo tee /etc/ld.so.conf.d/llama-cpp.conf
sudo ldconfig
```

Verified with `llama-server -v` and `ldconfig -p | grep llama`. If `build/` is ever wiped and reconfigured/rebuilt, the `.so` files regenerate at the same path, so this entry keeps working — re-run `sudo ldconfig` after any rebuild as a safety habit.

Alternatives considered and rejected: an `LD_LIBRARY_PATH` env var (session/service-scoped only, easy to forget in a new shell or future systemd unit); symlinking each `.so` individually into `/usr/local/lib` (same ongoing maintenance burden as the `ldconfig` fix, no real advantage).

# Pinning CPU cores

The benchmark and serving commands pin llama.cpp to a fixed number of CPU threads (`-t`) rather than letting it use every core, to get consistent, comparable benchmark numbers and leave headroom for the rest of the VM. See [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks#Key findings — text models (all on this VM's CPU, thread-pinned)|Benchmarks]] for the thread-scaling data this choice was based on (generation speed saturates around 3–4 threads on this hardware; prompt processing keeps scaling further). See [[Homelab/Platform/Local-AI-VM/Fedora/Cpu-configuration#CPU pinning and core isolation|Fedora: CPU pinning and core isolation]] for the VM-level vCPU/`affinity` pinning this thread count was chosen alongside.

# Gotchas

## Incremental rebuild silently linked a debug-flavored binary

The very first `llama-bench` run reported console warnings (`asserts enabled`, `debug build`) despite `CMakeCache.txt` correctly showing `CMAKE_BUILD_TYPE=Release` — numbers from that run were roughly 8x slower than the real baseline and had to be discarded.

**Root cause:** CMake's default Linux generator (`make`) decides whether to recompile a source file based on file timestamps, not on whether compiler flags changed. An earlier configure (before `-DCMAKE_BUILD_TYPE=Release` was set) had already compiled some object files with debug/assert flags; re-running `cmake -B build` with `Release` correctly updated the cache and `flags.make`, but `make` didn't consider those already-compiled `.o` files stale, so they were linked as-is into the final binary alongside newly-recompiled Release object files — a binary that reports `Release` in its build cache while still containing debug-compiled code.

**Fix:** after changing a significant CMake option (build type, native flags, etc.) on an existing build directory, do a full `rm -rf build` + reconfigure + rebuild rather than an incremental one, to guarantee every object file is compiled under the current, correct flags.

**Not the same cause everywhere:** the same `asserts enabled, performance may be affected` warning also shows up on the [[Daily-driver-laptop/Llama.cpp/Installation#The "asserts enabled" warning — root cause confirmed|Gigabyte laptop]], but for a different, unrelated reason — that machine uses a prebuilt Arch package whose `llama-cpp` PKGBUILD explicitly sets `-DCMAKE_BUILD_TYPE=None` (a deliberate upstream packaging choice, confirmed by reading the PKGBUILD directly), not a stale incremental build like this VM's.

## Distro-packaged `llama-bench`/`llama-cli` is the wrong build for this hardware

A precompiled `llama-bench` available via the distro's own package repo (separate from the self-built `./build/bin/llama-bench`) was tried as a comparison point. It reported `backend: ROCm` (AMD's GPU compute framework) and printed `ggml_cuda_init: failed to initialize ROCm: no ROCm-capable device is detected`, then produced drastically worse numbers (pp512: 5.12 t/s, tg128: 4.39 t/s) than the self-built CPU binary (~55-56 t/s pp512 / ~13 t/s tg128 at the same thread count). Passing `-ngl 0` (force zero GPU layers) did not fix it — the binary still attempted the ROCm backend regardless. This machine has no AMD GPU at all, so the distro package is fundamentally the wrong build variant here, independent of any flags — not pursued further. **All benchmark numbers for this project use the self-built binary.** (The Gigabyte laptop's distro-packaged install is a different situation: that machine actually has the matching NVIDIA/CUDA hardware the package targets, so the same "wrong backend for this hardware" problem doesn't apply there — see [[Daily-driver-laptop/Llama.cpp/Installation|its Installation page]].)

## Compiler warnings during the build are expected and safe to ignore

The build produces warnings such as `-Wdeprecated-enum-enum-conversion` and `-Wdeprecated-declarations` from llama.cpp's own upstream code (a bitwise OR between two different enum types; use of a deprecated filesystem API) — normal in a fast-moving open source project, and not something to chase down as long as the build completes and links (`Built target ...` lines print, no compiler errors). Only errors (the build stops, no binary produced) need fixing.

See [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks|Benchmarks]] for the resulting throughput numbers across models and thread counts, and [[Homelab/Platform/Local-AI-VM/Llama.cpp/Serving-as-a-service|Serving as a service]] for running it persistently. For the GPU-accelerated build on different hardware, see [[Daily-driver-laptop/Llama.cpp/Installation|the Gigabyte laptop's Llama.cpp bucket]].
