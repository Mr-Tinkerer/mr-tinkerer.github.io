---
title: "Pt 2, Enabling IOMMU"
description: Turning on IOMMU at the kernel/firmware level and confirming IOMMU groups.
tags:
  - Virtual_Machines
  - Hardware
---
# Pt 2, Enabling IOMMU

# Prerequisites
- [[Pt 1, Creating-the-vm|Pt 1, Creating the VM]]

---

## Kernel command line

AMD CPUs generally have this enabled already; Intel CPUs need `intel_iommu=on` added to the kernel command line.

- GRUB: append `intel_iommu=on` to `GRUB_CMDLINE_LINUX_DEFAULT` in `/etc/default/grub`, then run `grub-mkconfig -o /boot/grub/grub.cfg`.
- [Limine](https://github.com/Limine-Bootloader/Limine) (CachyOS's default bootloader): append the same value to `KERNEL_CMDLINE[default]` in `/etc/default/limine`, then run `limine-mkinitcpio`.

## Firmware (BIOS) settings

Reboot into the BIOS and look for `VT-d` (Intel) or `CBS` (AMD), plus an `IOMMU` option, if not already enabled. See [[Gigabyte-gaming-a16-ga6h-overview#Firmware / BIOS notes|this laptop's BIOS notes]] — its BIOS only exposes an "Intel (VMX) Virtualization Technology" toggle, no separate IOMMU option. If your BIOS has no visible IOMMU option either, boot into Linux and check with:

```
sudo dmesg | grep -i IOMMU
```

Seeing `DMAR: IOMMU enabled` confirms it's active regardless of the missing BIOS toggle.

## Checking IOMMU groups

The [Arch Wiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Ensuring_that_the_groups_are_valid) provides a script to list [[Windows-gaming-vm-glossary#IOMMU group|IOMMU groups]]:

```bash
#!/bin/bash
shopt -s nullglob
for g in $(find /sys/kernel/iommu_groups/* -maxdepth 0 -type d | sort -V); do
    echo "IOMMU Group ${g##*/}:"
    for d in $g/devices/*; do
        echo -e "\t$(lspci -nns ${d##*/})"
    done;
done;
```

A clean group contains only the GPU, its audio controller, and any other physical device that's expected to travel with it. If unrelated devices share the group, they must be passed through too (or try a different PCI slot for the GPU to land in a different group). On this laptop, the group is clean:

```
IOMMU Group 15:
	01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GB206M [GeForce RTX 5060 Max-Q / Mobile] [10de:2d19] (rev a1)
	01:00.1 Audio device [0403]: NVIDIA Corporation GB206 High Definition Audio Controller [10de:22eb] (rev a1)
```

---

Next: [[Pt 3, Isolating-the-gpu|Pt 3, Isolating the GPU]].
