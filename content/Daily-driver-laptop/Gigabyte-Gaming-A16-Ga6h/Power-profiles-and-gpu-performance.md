---
title: Gigabyte A16 Power Profiles vs. GPU Performance
description: The OS power profile on this laptop controls CPU behavior, but has an inverse, counterintuitive effect on GPU power headroom for GPU-bound workloads.
tags:
  - gigabyte-gaming-a16
  - power-management
  - gpu
---

This laptop's CPU and discrete GPU share a combined power/thermal budget rather than each having an independent ceiling (an Nvidia Max-Q / Dynamic Boost-style design). `powerprofilesctl`'s `performance`/`balanced`/`power-saver` profiles primarily govern **CPU** clocks/governor behavior — not GPU clocks directly — but on this laptop that has a large knock-on effect on the GPU.

This is a physical-hardware characteristic of this laptop model, not specific to any one piece of software — it would apply to any GPU-bound workload run under these power profiles, not just LLM inference. See [[Battery-management|Battery-management]] for the separate, unrelated automatic plug/unplug power scripting on this machine.

## The counterintuitive effect

For a GPU-bound workload, the CPU is mostly idle. Forcing the CPU governor to `performance` can still pull more of the laptop's *shared* power/thermal envelope toward the CPU (higher idle/base clocks, more aggressive boost on any CPU activity) — leaving *less* headroom for the GPU to boost, even though the CPU isn't the bottleneck. `power-saver` keeps CPU draw low, freeing more of the shared budget for the GPU, paradoxically *increasing* GPU-bound throughput.

## Confirmed directly via `nvidia-smi`

Rather than just inferring this from throughput numbers, I captured the GPU's own power limit directly with `nvidia-smi -q -d POWER` under each profile:

| Power Profile | GPU Power Limit | Default |
|---|---|---|
| `performance` | **45.85 W** | 50.00 W |
| `balanced` | 53.77 W | 50.00 W |
| `power-saver` | **60.86 W** | 50.00 W |

The GPU's power limit is set **inversely** to what the profile name suggests: `performance` caps the GPU below its own default, while `power-saver` raises it above default. The `performance` profile is evidently tuned around sustained CPU performance, and on this laptop's shared power design that comes directly at the GPU's expense — the opposite of what a user would reasonably expect from a profile named "performance" on a GPU-bound workload.

## Practical takeaway

- **GPU-bound workload** → use `power-saver`. It is not a trade-off against speed here — it's strictly faster.
- **CPU-bound workload** → use `performance`. Confirmed by a CPU-only llama.cpp run showing `performance` winning by ~54% (pp) and ~2.6x (generation) over `power-saver` — see [[Gigabyte-LLama-Benchmarks#Power profile vs. CPU-only throughput — Llama 3.1 8B, `-ngl 0`|the llama.cpp benchmarks]] for the full numbers.
- The effect size (up to ~2.6x on generation speed in the llama.cpp testing) is large enough that picking the wrong profile is a bigger performance loss than most other single tuning choice tested on this machine.

## Gotchas

- **This is real and machine-dependent, not a fixed rule.** Whether — and how strongly — this happens depends on this specific laptop's power-delivery/thermal design, the GPU's boost algorithm (Nvidia Dynamic Boost 2.0 explicitly shares budget between CPU/GPU on supported laptops), and how the power-profile daemon maps its profiles to actual clock behavior. Don't assume the same numeric power limits, or even the same profile-to-budget mapping, transfer to a different laptop model — the *pattern* (an OS "performance" profile potentially throttling GPU headroom) generalizes, but always re-measure on the specific machine.
- Confirm rather than assume: capture actual GPU power draw/limit and clocks (`nvidia-smi -q -d POWER,CLOCK`, or `nvidia-smi --query-gpu=power.draw,clocks.sm,clocks.mem --format=csv -l 1`) during both profiles for the workload in question, rather than inferring from throughput alone.

**Open items:**
- Whether the GPU power limit can be adjusted independently of the OS power profile (e.g. `nvidia-smi -pl <watts>`, if permitted on this laptop/driver), to combine `performance` CPU behavior with `power-saver`-level GPU headroom.
- Whether this is standard Gigabyte/Nvidia Advanced Optimus behavior for this model, or a CachyOS `power-profiles-daemon` mapping quirk specific to this install — check its config for how each profile is defined on this system.
