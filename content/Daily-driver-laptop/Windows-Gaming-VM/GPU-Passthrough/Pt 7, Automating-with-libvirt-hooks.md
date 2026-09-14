---
title: "Pt 7, Automating with Libvirt Hooks"
description: Wiring the start/stop driver-switching scripts to run automatically whenever the VM boots or shuts down.
tags:
  - Virtual_Machines
  - Bash_Script
---
# Pt 7, Automating with Libvirt Hooks

# Prerequisites
- [[Pt 1, Creating-the-vm|Pt 1, Creating the VM]]
- [[Pt 2, Enabling-iommu|Pt 2, Enabling IOMMU]]
- [[Pt 3, Isolating-the-gpu|Pt 3, Isolating the GPU]]
- [[Pt 4, Attaching-the-gpu|Pt 4, Attaching the GPU to the VM]]
- [[Pt 5, Setting-up-looking-glass|Pt 5, Setting up Looking Glass]]
- [[Pt 6, Dynamic-driver-switching|Pt 6, Dynamic driver switching]]

---

## Why not just run the scripts manually in a terminal

Running the stop/release script from inside a Niri terminal doesn't work cleanly: the script's own steps eventually restart Niri (to make it stop grabbing the GPU), which kills every open window — including the terminal running the script itself.

## Libvirt hooks

[[Windows-gaming-vm-glossary#Libvirt hook|Libvirt hooks]] let a script at `/etc/libvirt/hooks/qemu` run automatically on VM lifecycle events, independent of any user session. After I confirmed the hook fires (tested by having it `touch` a marker file and confirming it appeared after `systemctl restart libvirtd`), I wired it to the real start/stop scripts described in [[Pt 6, Dynamic-driver-switching#Final scripts|Pt 6]]: the hook checks whether the VM lifecycle event is for the `win10` VM specifically, then runs the starting script on `prepare`/`begin` (VM boot) and the stopping script on `release`/`end` (VM shutdown). The hook script itself is maintained at [qemu](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Daily%20Driver%20Laptop/GPU%20Passthrough/qemu) rather than pasted here.

Restarting `libvirtd` picks up the hook. From then on, starting/stopping the `win10` VM automatically drives the GPU between the host and the guest with no manual scripting.

## What the start script does before switching drivers

Before touching drivers, the start script stops `nvidia-powerd`, disables the GPU's audio device in WirePlumber, and gets a list of processes still holding `/dev/nvidia*` (via `lsof`, filtered to exclude Niri itself since that's handled separately). If anything is still using the GPU, it sends me a persistent, self-updating desktop notification asking me to close those programs, and polls every second until the list is empty before proceeding to unbind/rebind and restart Niri with the GPU-disabling config included.

## End-to-end result

![[automatic-gpu-passthrough-demo.gif]]
*The GPU automatically moving from host to VM and back as the VM starts and stops, driven entirely by the libvirt hook — the fully automated result this whole sequence was built toward.*

The one remaining rough edge: Niri still has to restart to stop grabbing the GPU, which closes any open windows. There isn't currently a way around that on Niri; a KDE-based approach to avoid it is a possible future improvement, not yet implemented.

## Reference implementation

The finished scripts referenced throughout this sequence (`starting_GPU_VM.sh`, `stopping_GPU_VM.sh`, and the qemu hook script above) are maintained at: [GPU-Passthrough](https://github.com/Mr-Tinkerer/Project-Dump/tree/main/Daily%20Driver%20Laptop/GPU%20Passthrough).
