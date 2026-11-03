---
title: Fedora CPU Configuration on Local-AI VM (Fedora)
description: How the Local AI VM's virtual CPU and core pinning are configured at the guest-OS/VM level, independent of any service running on it.
tags:
  - Homelab
  - Fedora
---

## Overview

This page documents the Local AI VM's virtual CPU configuration in Proxmox — settings that would be true even without llama.cpp installed, so they belong with the Fedora guest OS rather than with llama.cpp's build page. That page links back here for the "why" behind the CPU shape it benchmarks against.

## CPU Type: Host Passthrough

The VM's CPU type was set to `host` passthrough rather than a specific x86-64 microarchitecture level. The guest sees the physical CPU's exact flags.

### Decision Rationale

Host mode was chosen since this VM isn't expected to migrate to different physical hardware, prioritizing maximum performance over portability.

## CPU Pinning and Core Isolation

The VM's vCPUs originally mapped 1:1 to all of the host's physical cores, meaning it could claim 100% of host CPU under load and starve other VMs on the same Proxmox host regardless of their own vCPU allocations.

Based on llama.cpp's thread-sweep data, generation speed shows diminishing returns past ~3–4 threads. A small amount of generation speed (~5%, measured) was sacrificed in exchange for guaranteeing some physical cores stay available to other VMs.

### Implementation

- **Reduced vCPU allocation** — Fewer `cores:` than the host's physical core count
- **CPU pinning** — Pinned the VM to a specific range of physical cores via Proxmox `affinity`

### Physical Core Mapping Verification

On a Hyperthreading/SMT-enabled host, two logical CPU IDs can be two threads of the same physical core, in which case pinning to them yields far fewer real cores than expected. The physical-core-to-logical-CPU mapping was checked first:

```bash
lscpu -e
```

### Failed cpuset Configuration

Adding a `cpuset: <range>` key directly to the VM's `/etc/pve/qemu-server/<vmid>.conf` was accepted and written to the file, but had no actual effect — `cpuset` is not a real/recognized Proxmox VM config key (at least on this Proxmox version). Proxmox silently stores unrecognized keys in the `.conf` file without validating or applying them, so it looked configured but was never enforced by the kernel/cgroup layer.

Verification showed all cores instead of the intended subset:

```bash
grep Cpus_allowed_list /proc/$(cat /var/run/qemu-server/<vmid>.pid)/task/*/status | sort -u
```

### Correct Fix: Affinity

Use `affinity`, the real supported Proxmox 8.x+ option, applied via cgroups/taskset:

```bash
qm set <vmid> --affinity <range>
```

Unlike the `cpuset` misconception, this can be applied to a running VM without a full stop/start, since it acts on the already-running QEMU process directly.
