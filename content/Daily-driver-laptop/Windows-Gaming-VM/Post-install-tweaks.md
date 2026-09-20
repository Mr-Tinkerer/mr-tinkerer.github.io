---
title: Gaming VM Post-Install Tweaks
description: CPU pinning, faster VirtIO drives, and Looking Glass port/hotkey tweaks I applied to the Windows gaming VM after install.
tags:
  - Virtual_Machines
  - QEMU
  - Performance
---
# Gaming VM Post-Install Tweaks

A few tweaks I applied to the VM after the base install, on top of the [[GPU-Passthrough/Pt 1, Creating-the-vm|GPU passthrough setup]], to get better and more predictable performance.

## CPU pinning

I pin the VM's vCPUs to specific host cores so it has full, exclusive control over them instead of competing with the host scheduler for whichever core happens to be free.

1. List the host's CPU topology:
   ```bash
   lscpu -e
   ```
   On my [[Gigabyte-gaming-a16-ga6h-overview|Gigabyte Gaming A16]]'s 13th Gen Intel Core i7-13620H, cores 0–5 are the performance cores (with hyperthreading) and 6–9 are the efficiency cores.
2. I pin cores 0–11 (all performance-core threads) to the VM.
3. In the VM's CPU configuration, I enable `host-passthrough` and manually set the topology to match: 1 socket, 6 cores, 2 threads.
4. Then I add the pinning itself directly in the VM's raw XML:
   ```xml
   <vcpu placement="static">12</vcpu>
   <cputune>
     <vcpupin vcpu="0" cpuset="0"/>
     <vcpupin vcpu="1" cpuset="1"/>
     <vcpupin vcpu="2" cpuset="2"/>
     <vcpupin vcpu="3" cpuset="3"/>
     <vcpupin vcpu="4" cpuset="4"/>
     <vcpupin vcpu="5" cpuset="5"/>
     <vcpupin vcpu="6" cpuset="6"/>
     <vcpupin vcpu="7" cpuset="7"/>
     <vcpupin vcpu="8" cpuset="8"/>
     <vcpupin vcpu="9" cpuset="9"/>
     <vcpupin vcpu="10" cpuset="10"/>
     <vcpupin vcpu="11" cpuset="11"/>
   </cputune>
   ```

## Faster drives

- I set the drive's **Discard mode** to `unmap` for SSD TRIM support — see [[Virtual-disks-reference#7. Discard / TRIM|the virtual disks reference]] for why this matters and its caveats.
- I set the drive's bus type to **VirtIO** for better performance over SATA/IDE. This requires the [[Libvirt-glossary#VirtIO drivers|VirtIO drivers]] to be installed in the Windows guest before the drive is detected, plus the VirtIO Serial Controller hardware added to the VM. Installing from the VirtIO ISO during Windows setup only gets the SCSI driver in; the rest still needs installing from inside the already-installed OS.

## Looking Glass port and hotkey

- I set the VM's SPICE port to `6769` instead of leaving it on the default, so another VM on the same host can't accidentally hijack [[Windows-gaming-vm-glossary#Looking Glass|Looking Glass]]'s connection.
- I changed Looking Glass's menu/hotkey from the default Scroll Lock to backslash, since this laptop's keyboard has no Scroll Lock key.
- The flags I use to launch the client: `-m KEY_BACKSLASH -p 6769`.
