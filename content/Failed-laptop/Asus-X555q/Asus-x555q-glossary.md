---
title: Asus X555q Glossary
description: Terms used across the Asus X555q bucket.
tags:
  - Glossary
---
## APU

Accelerated Processing Unit — a chip combining a CPU and GPU on one die, instead of a separate discrete GPU. This laptop's AMD A12-9720P is an APU, which matters here only as a spec: I'm running it headless as a Proxmox/NAS box, so the integrated GPU sits unused and all that matters is the CPU side's core count and clock speed. See [Specs and Benchmarks](https://www.notebookcheck.net/AMD-A12-9720P-SoC-Benchmarks-and-Specs.234448.0.html).

## WPA2 Enterprise

A Wi-Fi security mode that authenticates each user individually (via a RADIUS server) instead of a single shared password, commonly used on institutional networks. See [Wikipedia:WPA-Enterprise](https://en.wikipedia.org/wiki/Wi-Fi_Protected_Access#WPA-Enterprise).

## GNOME

The default desktop environment on Ubuntu and most other mainstream Linux distros. It matters here because its Wi-Fi connection UI has no fields for WPA2 Enterprise credentials (PEAP, inner authentication, identity/password) — so a GNOME-based live USB can't join my dorm's enterprise network at all, which is what pushed me to KDE just to get initial connectivity. See [gnome.org](https://www.gnome.org/).

## KDE

An alternative Linux desktop environment (used here via Kubuntu), whose Wi-Fi connection dialog supports WPA2 Enterprise fields that GNOME's does not. See [kde.org](https://kde.org/).

## ifupdown

Debian's traditional network interface management tool, configured via `/etc/network/interfaces` rather than through `NetworkManager`'s UI/daemon model. Both Debian and Proxmox (which is Debian-based) default to it. It matters here because `NetworkManager` and `ifupdown` can't manage the same interface at once — I had to make sure `ifupdown` was in control (and `NetworkManager` disabled) before `wpa_supplicant` and my `/etc/network/interfaces` entries would actually take effect on the Wi-Fi dongle. See the [Debian Wiki: Network Configuration](https://wiki.debian.org/NetworkConfiguration).

## OS-agnostic test

A diagnostic question used to decide whether a problem belongs to hardware or software: if a different OS or software on the same physical hardware, in the same environment, would hit the same problem and need the same fix, the root cause is hardware/environment — not the specific software.

## NAS

Network Attached Storage — a device on the local network dedicated to storing and sharing files, so any machine on the network can read/write to it instead of each machine keeping its own separate copy of data. This is the whole point of the second life I gave this laptop: after the Proxmox experiment, I repurposed it to run OpenMediaVault as a NAS VM so my other machines (like my daily driver) could mount a shared drive and back up to it over the network. See [Wikipedia: Network-attached storage](https://en.wikipedia.org/wiki/Network-attached_storage).

## OpenMediaVault

A Debian-based NAS operating system providing a web UI for storage, file-sharing, and user management. See the [OpenMediaVault documentation](https://docs.openmediavault.org/).

## BTRFS

A Linux filesystem supporting snapshots, self-healing (checksums), and transparent compression. See the [BTRFS documentation](https://btrfs.readthedocs.io/en/latest/Introduction.html). See [[Homelab/Software/OpenMediaVault/Btrfs-reference|my deeper BTRFS reference]] in the Homelab domain for allocation model, balance/scrub, and quota details that apply here too.

## NFS

Network File System — a file-sharing protocol originating on Unix systems, using a Unix-native permission model. See [Wikipedia: Network File System](https://en.wikipedia.org/wiki/Network_File_System).

## SMB/CIFS

Server Message Block (and its CIFS dialect) — a file-sharing protocol originating on DOS/Windows and native to Windows file sharing. See [AWS: NFS vs. SMB](https://aws.amazon.com/compare/the-difference-between-nfs-smb/).

## ACL

Access Control List — a per-file/folder list of user or group permissions that is separate from, and can override, the classic Unix owner/group/other permission bits. See the [Arch Wiki: ACL](https://wiki.archlinux.org/title/Access_Control_Lists).

## autofs

A Linux service that mounts filesystems on demand when they're accessed and unmounts them after a period of inactivity, instead of mounting everything unconditionally at boot. See the [Linux kernel autofs documentation](https://www.kernel.org/doc/html/latest/filesystems/autofs.html).

## BorgBackup

A deduplicating, compressing, and encrypting backup tool for Unix-like systems (no Windows support). I use it to back up my daily driver's home directory onto the NAS share this laptop hosts — deduplication matters a lot here since repeated backups of the same mostly-unchanged home directory only store the data that actually changed, which is what keeps the backup repo's size manageable over many runs. See [borgbackup.org](https://www.borgbackup.org/).
