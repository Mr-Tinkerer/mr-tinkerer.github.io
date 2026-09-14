---
title: Fedora Glossary
description: Terms specific to the Fedora guest OS running on the Local AI VM.
tags:
  - Homelab
  - Fedora
---
## host CPU passthrough

A Proxmox VM CPU-type setting where the guest is presented with the physical host CPU's exact instruction set and feature flags, rather than a generic/portable virtual CPU profile. It gives the best possible performance for CPU-bound workloads (like local LLM inference) at the cost of portability — a VM configured this way can't safely be live-migrated to a host with a different physical CPU, since the guest OS may have detected and started relying on instruction-set features (like AVX2) that the new host doesn't have. I use it here because this VM isn't expected to move to different hardware.

## x86-64 microarchitecture level

A standardized tier (`x86-64-v1` through `x86-64-v4`) that groups CPU instruction-set extensions into portable baselines — for example `x86-64-v3` guarantees AVX2, FMA, and BMI2 are present. Proxmox lets a VM's CPU type be pinned to one of these levels instead of to the exact host CPU, which is the safer choice when a VM might migrate between hosts with different physical CPUs, since any host new enough to claim that level is guaranteed to support everything it requires. See the [x86-64 microarchitecture levels reference](https://en.wikipedia.org/wiki/X86-64#Microarchitecture_levels).

## AVX2

Advanced Vector Extensions 2, a CPU instruction set (present on Intel/AMD CPUs since roughly 2013/2015) that accelerates the kind of vectorized floating-point math CPU-based LLM inference leans on heavily. Whether a VM's guest OS actually gets to use AVX2 depends on how its CPU type is configured — see [[Homelab/Platform/Local-AI-VM/Fedora/Fedora-glossary#host CPU passthrough|host CPU passthrough]] above.
