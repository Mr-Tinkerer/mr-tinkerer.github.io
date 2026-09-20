---
title: libvirt glossary
description: Terms used in the libvirt snapshot and VMware migration pages.
tags:
  - libvirt
  - glossary
---
# Glossary

## QEMU
A machine emulator/virtualizer that runs VMs on top of a Linux distro, emulating CPU, storage, network, and other virtual hardware for the guest. On its own QEMU is a Type 2-style emulator; paired with [[Libvirt-glossary#KVM|KVM]] it gets direct hardware access through the kernel, which is what makes GPU passthrough practical (see [[Windows-gaming-vm-glossary#VFIO|VFIO]]). I also use its `qemu-img` tool to inspect, convert, and repair disk images throughout this bucket. See [qemu.org](https://www.qemu.org/) / [Wikipedia: QEMU](https://en.wikipedia.org/wiki/QEMU).

---

## KVM
Kernel-based Virtual Machine — a virtualization module built into the Linux kernel that lets QEMU bypass the host OS and access hardware directly, turning QEMU into a de facto Type 1 hypervisor. See [Wikipedia: KVM](https://en.wikipedia.org/wiki/Kernel-based_Virtual_Machine).

---

## libvirt
A toolkit that manages QEMU/KVM VMs and stores their configuration as XML instead of long command lines. I interact with it mostly through [[Libvirt-glossary#Virt Manager|Virt Manager]]'s GUI, but drop into raw XML editing or `virsh` directly for things the GUI doesn't expose (IVSHMEM devices, CPU pinning, libvirt hooks, snapshots). It also runs its own DHCP/DNS/NAT stack for each virtual network it creates — see [[Libvirt-networking-and-firewall]] for how that interacts with the host firewall. See [libvirt.org](https://libvirt.org/).

---

## Virt Manager
A GUI front-end for libvirt (and by extension QEMU/KVM) that I use to create VMs, fix their CPU topology, attach PCI devices for GPU passthrough, and edit the VM's underlying XML directly (e.g. to add the IVSHMEM device for Looking Glass) when the GUI alone isn't enough. See [virt-manager.org](https://virt-manager.org/).

---

## VirtIO drivers
A set of paravirtualized Windows drivers (disk, network, memory ballooning) that let a Windows guest talk to QEMU's virtual devices efficiently. Maintained by the Fedora Project. Linux guests include the equivalent drivers in the kernel. For how they interact with disk bus and controller choices, see [[Virtual-disks-reference]]. See [Proxmox: Windows VirtIO Drivers](https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers).

---

## QEMU Guest Agent
A guest-side daemon that exchanges information with the host — clean shutdowns, filesystem freeze for snapshots/backups, and automatic guest display resizing. I install it alongside the [[Libvirt-glossary#VirtIO drivers|VirtIO drivers]] right after Windows setup (see [[GPU-Passthrough/Pt 1, Creating-the-vm|Pt 1]]) since libvirt/Virt Manager can't cleanly shut the VM down or resize its display without it running inside the guest. The filesystem freeze is what `--quiesce` uses in [[Libvirt-external-snapshots]]. See [Proxmox: qemu-guest-agent](https://pve.proxmox.com/wiki/Qemu-guest-agent#Windows).

---

## nftables
The Linux kernel's current packet-filtering/NAT framework (successor to iptables), organized into tables and chains hooked into points like `input`/`forward`/`output`. libvirt, UFW, and any manual rules I write all install their own independent tables into the same nftables framework — see [[Libvirt-networking-and-firewall]] for why that independence matters. See [Arch Wiki: nftables](https://wiki.archlinux.org/title/Nftables).

---

## UFW
"Uncomplicated Firewall" — a policy front-end over nftables/iptables-nft. It manages its own tables and chains independently of anything I write by hand in `/etc/nftables.conf`, which is the root cause of a libvirt DHCP/DNS failure documented in [[Libvirt-networking-and-firewall]]. See [Ubuntu: UFW](https://help.ubuntu.com/community/UFW).

---

## dnsmasq
A lightweight DHCP/DNS server; libvirt runs one instance per virtual network, bound to that network's bridge interface, to hand out leases and resolve names for VMs on it. See [dnsmasq project page](https://thekelleys.org.uk/dnsmasq/doc.html).

---

## SPICE
A remote-display protocol QEMU can expose a VM's console over, viewed with `virt-viewer`/`remote-viewer` or through Virt Manager's built-in console. It's the display path I use before/instead of [[Windows-gaming-vm-glossary#Looking Glass|Looking Glass]] — see [[Spice-auto-resize-fix]] for a Wayland-specific scaling bug I hit with it. See [spice-space.org](https://www.spice-space.org/).

---

## External snapshot

A libvirt snapshot type that freezes the current disk image and creates a new [[Libvirt-glossary#Overlay|overlay]] file for all later writes. The frozen file keeps the disk as it was at snapshot time. This matters here because libvirt can create external snapshots but cannot delete or merge them, so I clean them up by hand. See [[Libvirt-external-snapshots]] for how the layers line up.

Reference: [libvirt snapshot XML format](https://libvirt.org/formatsnapshot.html)

---

## Backing file and backing chain

A [[Libvirt-glossary#QCOW2|QCOW2]] image can name another image as its backing file. Reads that the top image cannot answer fall through to the backing file. Following those links from the active image down to the base gives the backing chain. Almost every snapshot problem I hit came down to which file in the chain I was looking at.

Reference: [libvirt backing chain management](https://libvirt.org/kbase/backing_chains.html)

---

## Overlay

A QCOW2 image that stores only the writes made since its [[Libvirt-glossary#Backing file and backing chain|backing file]] was frozen. It is small when created and grows as the guest writes. I never boot directly from a frozen backing file. I put a fresh overlay on top of it first.

Reference: [qemu-img create](https://www.qemu.org/docs/master/tools/qemu-img.html)

---

## Flattening

Collapsing a whole [[Libvirt-glossary#Backing file and backing chain|backing chain]] into one standalone image with `qemu-img convert`. The result has no dependencies on older overlays. I flatten after a recovery, and I avoid flattening during a VMware migration when I want to keep a chain.

Reference: [qemu-img convert](https://www.qemu.org/docs/master/tools/qemu-img.html)

---

## Disk-only snapshot

A snapshot that records disk state and no RAM or device state. It restores like a power cut. I use it to avoid the virtiofs restore failure described in [[Libvirt-external-snapshot-recovery]].

Reference: [virsh snapshot commands](https://libvirt.org/manpages/virsh.html)

---

## Crash-consistent

A disk state equal to what the disk would hold if power were cut at that instant. Journaling filesystems such as NTFS, ext4, and xfs replay their journal and recover from it. Snapshots without RAM state and offline copies of a running disk are crash-consistent unless the guest agent quiesces the filesystem first.

Reference: [libvirt snapshot XML format](https://libvirt.org/formatsnapshot.html)

---

## QCOW2

The QEMU disk image format I use for VM disks. It supports backing files, snapshots, compression, and sparse allocation. VMware VMDK disks get converted to it during migration.

Reference: [QCOW2 image format specification](https://www.qemu.org/docs/master/interop/qcow2.html)

---

## virtiofs

A shared-folder mechanism between a KVM host and a guest. QEMU delegates the filesystem work to a separate `virtiofsd` process. That process holds backend state QEMU cannot reload from a saved RAM snapshot, which is why live snapshots of a VM with a virtiofs device fail to restore.

Reference: [virtio-fs project](https://virtio-fs.gitlab.io/)

---

## libvirt snapshot metadata

The `domainsnapshot` XML records libvirt keeps for named snapshots. It is separate from the QCOW2 backing chain on disk. A working chain does not create a named snapshot, and a named snapshot does not guarantee a working chain. I check both.

Reference: [libvirt snapshot XML format](https://libvirt.org/formatsnapshot.html)

---

## VMDK

The VMware virtual disk format. A VMDK disk is a small descriptor file plus one or more extent files that hold the data. The extents are pieces of one disk, so I convert the descriptor and never the extents.

Reference: [VMDK format specification (libvmdk project)](https://github.com/libyal/libvmdk/blob/main/documentation/VMWare%20Virtual%20Disk%20Format%20(VMDK).asciidoc)

---

## VMware VM files

The files in a VMware VM directory: `.vmx` (hardware configuration), `.vmsd` (snapshot names and relationships), `.vmsn` (snapshot state, including memory for running VMs), and `.nvram` (firmware state). I use `.vmx` and `.vmsd` as references when rebuilding the VM. None of them is a portable libvirt file.

Reference: [VMDK format specification (libvmdk project)](https://github.com/libyal/libvmdk/blob/main/documentation/VMWare%20Virtual%20Disk%20Format%20(VMDK).asciidoc)
