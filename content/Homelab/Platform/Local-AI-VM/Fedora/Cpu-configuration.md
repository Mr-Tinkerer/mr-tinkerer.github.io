---
title: CPU configuration
description: How I configured the Local AI VM's virtual CPU and core pinning at the guest-OS/VM level, independent of any service running on it.
tags:
  - Homelab
  - Fedora
---
# Why this lives here, not under llama.cpp

These are facts about how the Local AI VM's virtual CPU itself is configured in Proxmox — they'd be true even if I hadn't installed llama.cpp at all, so they belong with the Fedora guest OS rather than with [[Homelab/Platform/Local-AI-VM/Llama.cpp/Build-and-cpu-tuning|llama.cpp's build page]]. That page links back here for the "why" behind the CPU shape it benchmarks against.

# CPU type: host passthrough

I used Claude to help work through this whole local-AI setup, and it was Claude that first told me to check whether the VM had AVX2 support at all — it didn't. Proxmox lets a VM's CPU type be set either to a specific [[Homelab/Platform/Local-AI-VM/Fedora/Fedora-glossary#x86-64 microarchitecture level|x86-64 microarchitecture level]] (e.g. `x86-64-v3`, which guarantees AVX2) or to [[Homelab/Platform/Local-AI-VM/Fedora/Fedora-glossary#host CPU passthrough|`host` passthrough]] (the guest sees the physical CPU's exact flags). I chose `host` mode here, since this VM isn't expected to migrate to different physical hardware, prioritizing maximum performance over portability.

# CPU pinning and core isolation

This VM's vCPUs originally mapped 1:1 to all of the host's physical cores, meaning it could claim 100% of host CPU under load and starve other VMs on the same Proxmox host regardless of their own vCPU allocations. Based on llama.cpp's own thread-sweep data (see [[Homelab/Platform/Local-AI-VM/Llama.cpp/Benchmarks#Key findings — text models (all on this VM's CPU, thread-pinned)|Benchmarks]] — generation speed shows diminishing returns past ~3–4 threads), I decided to sacrifice a small amount of generation speed (~5%, measured) in exchange for guaranteeing some physical cores stay available to other VMs.

**Implementation:**
- I reduced the vCPU allocation (fewer `cores:` than the host's physical core count).
- I pinned the VM to a specific range of physical cores via Proxmox `affinity`.

**Determining which logical CPUs to pin to matters** — on a Hyperthreading/SMT-enabled host, two logical CPU IDs can be two threads of the *same* physical core, in which case pinning to them yields far fewer real cores than expected. I checked the physical-core-to-logical-CPU mapping first:

```bash
lscpu -e
```

**First attempt failed silently:** adding a `cpuset: <range>` key directly to the VM's `/etc/pve/qemu-server/<vmid>.conf` was accepted and written to the file, but had **no actual effect** — `cpuset` is not a real/recognized Proxmox VM config key (at least on this Proxmox version). Proxmox silently stores unrecognized keys in the `.conf` file without validating or applying them, so it *looked* configured but was never enforced by the kernel/cgroup layer. I verified this via:

```bash
grep Cpus_allowed_list /proc/$(cat /var/run/qemu-server/<vmid>.pid)/task/*/status | sort -u
# showed all cores, not the intended subset
```

**Correct fix:** use `affinity`, the real supported Proxmox 8.x+ option, applied via cgroups/`taskset`:

```bash
qm set <vmid> --affinity <range>
```

Unlike the `cpuset` misconception, this can be applied to a running VM without a full stop/start, since it acts on the already-running QEMU process directly. I verified it the same way as above, now correctly showing the pinned range.
