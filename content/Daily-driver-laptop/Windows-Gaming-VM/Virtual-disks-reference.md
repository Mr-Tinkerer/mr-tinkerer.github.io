---
title: QEMU/KVM Virtual Disks Reference
description: My reference for everything that determines how a QEMU/KVM virtual disk behaves — bus type, controller, file format, and performance/safety settings.
tags:
  - Virtual_Machines
  - QEMU
  - Storage
---
# QEMU/KVM Virtual Disks Reference

I put this together as my own reference for Virt-Manager (and Proxmox) virtual disks — everything that determines how a disk behaves: the bus it's attached to, the controller emulating that bus, the file format backing it, and the settings that control performance, space usage, and data safety. I used it when setting up the [[Windows-gaming-vm-glossary#VirtIO drivers|VirtIO]]-backed drives for the gaming VM (see [[Post-install-tweaks]]), and keep it here since it applies to any QEMU/KVM VM, not just that one.

---

## 1. Foundational concept: emulation vs. paravirtualization

Nearly every choice below comes down to one tradeoff, worth understanding first since it explains *why* the various options behave the way they do.

**Emulation** — the hypervisor pretends to *be* a specific, real piece of hardware, down to register-level behavior. The guest OS has no idea it's virtualized; it uses the same driver it would on bare metal. Every I/O operation gets trapped, simulated in software, and returned to the guest.
- Pros: maximum compatibility — any OS with a driver for that real hardware just works.
- Cons: slower — every operation needs the host CPU to trap, interpret, and simulate.
- Examples: IDE, SATA/AHCI, LSI 53C895A, LSI 53C810, MegaRAID SAS 8708EM2.

**Paravirtualization** — the guest OS *knows* it's virtualized and cooperates directly with the hypervisor through a purpose-built interface, not a simulation of real hardware. Guest and host share memory regions (VirtIO's *virtqueues*) where I/O requests land directly in a format the host already understands, skipping the trap-and-simulate cycle.
- Pros: much faster, drastically less CPU overhead per operation.
- Cons: needs a driver that understands the paravirtualized protocol (inbox on modern Linux; needs the `virtio-win` package on Windows).
- Examples: VirtIO Block, VirtIO SCSI, VirtIO SCSI Single, VMware PVSCSI.

| | Emulation | Paravirtualization |
|---|---|---|
| Guest awareness | Thinks it's real hardware | Knows it's virtualized |
| Driver needed | Whatever the real chip needs (often inbox) | Purpose-built (VirtIO drivers) |
| Performance | Slower | Faster |
| Compatibility | Extremely broad | Requires explicit driver support |

---

## 2. Bus types

The bus type determines what interface the guest OS thinks the disk is plugged into — the first fork in the road for compatibility and performance.

- **IDE (PATA)** — oldest option, hard limit of 4 devices per VM. Maximum compatibility (DOS, Windows 9x/2000/XP, ancient Linux/BSD), zero drivers needed, but worst performance, no hot-plug, essentially no TRIM/discard. Best for retro installs or the CD-ROM/ISO slot.
- **SATA (AHCI)** — emulates a standard AHCI controller. Universal inbox compatibility, noticeably slower than VirtIO. Good for legacy OSes predating VirtIO, or as a safe bootstrap bus during installs/migrations.
- **USB** — emulates a USB mass-storage device; useful for simulating removable storage, poor performance, rarely used as a primary disk.
- **SCSI** — routes through an emulated or paravirtualized SCSI controller (Section 3); the bus itself is just "SCSI," the controller determines performance/compatibility.
- **VirtIO (`virtio-blk`)** — paravirtualized block device, best raw performance and lowest CPU overhead. Needs VirtIO drivers (inbox on Linux, `virtio-win` on Windows). Shows up as `/dev/vda` rather than `/dev/sda` on Linux. Default choice for most Linux VMs and Windows VMs once drivers are installed.

| Bus Type | Compatibility | Performance | Driver Needed? |
|---|---|---|---|
| IDE | Extreme (ancient OSes) | Worst | None |
| SATA | Very broad | Moderate | None (inbox) |
| USB | Removable-media use case | Poor | None (inbox) |
| SCSI (via controller) | Depends on controller | Depends on controller | Depends on controller |
| VirtIO | Requires VirtIO support | Best | Inbox (Linux) / `virtio-win` (Windows) |

---

## 3. SCSI controllers

Choosing "SCSI" as the bus just tells the disk to attach to a SCSI bus — the **controller** is the virtual HBA chip QEMU emulates, and it matters more than the bus choice alone.

- **LSI 53C895A ("LSI Logic")** — emulation of a late-90s Symbios/LSI Ultra2 chip; Proxmox's default. Less likely to be fingerprinted as a VM, but fully emulated (slower).
  > ⚠️ **Windows-specific gotcha:** this emulates the *parallel* SCSI variant. Microsoft's LSI Logic Parallel SCSI driver hasn't been actively maintained since Windows favored LSI Logic *SAS* starting with Vista/Server 2008 — so **Windows 10/11 has no reliable inbox driver for this controller at all**, despite older docs (written with XP/2003 in mind) claiming "no drivers needed." If a Windows 10+ guest reports a missing controller driver on a SCSI-bus disk, this is almost certainly why; the fix is VirtIO SCSI. **Linux is unaffected** — the in-kernel `sym53c8xx_2` driver covers the whole 53C8XX family and is auto-loaded by every mainstream distro (the one edge case being a custom-built minimal kernel/initrd that strips it out).
- **LSI 53C810** — even older, extreme legacy compatibility only, with real reported reliability issues. No upside over the 53C895A today.
- **MegaRAID SAS 8708EM2** — emulates a real hardware RAID card. Slightly better performance than 53C895A in some cases, but conceptually mismatched for a simple single-disk VM, and needs a driver not available on every guest OS.
- **VirtIO SCSI** — paravirtualized; one controller instance handles many disks (Proxmox cites up to 16, more in practice), all sharing one thread. Much better performance than any LSI/MegaRAID emulation, proper TRIM/discard. Recommended default for Linux since Proxmox VE 4.3.
- **VirtIO SCSI Single** — same protocol, but one controller *per disk*. This is what unlocks [[#5. The IOThread option|IOThread]] — each disk gets its own thread, measurably better under concurrent multi-disk I/O. Proxmox's default (with IOThread) since PVE 7.3. Without IOThread enabled, the difference versus plain VirtIO SCSI is minimal.
- **VMware PVSCSI** — QEMU's emulation of VMware's paravirtualized controller; only useful as a bridge during a VMware→KVM migration (keeping the source VM's PVSCSI disks bootable without an in-guest storage reconfig), switch to VirtIO SCSI afterward.

| Controller | Best for |
|---|---|
| **VirtIO SCSI Single + IOThread** | Best overall — modern Linux/Windows, especially multi-disk or I/O-heavy VMs |
| **VirtIO SCSI** | Solid simpler default when per-disk IOThread tuning isn't needed |
| **LSI 53C895A** | Legacy fallback; works great on Linux, unreliable on modern Windows |
| **LSI 53C810** | Rarely justified |
| **MegaRAID SAS 8708EM2** | Niche compatibility testing only |
| **VMware PVSCSI** | Stopgap during VMware → KVM migration only |

---

## 4. Expected performance: SATA vs. SCSI/VirtIO

Paravirtualization *should* consistently outperform SATA emulation, since VirtIO skips most of the per-operation trap-and-simulate cost. In practice, real benchmarks are messier than that framing suggests — the direction is usually right, but the size of the gap varies a lot by workload and storage backend:

- A controlled Windows guest benchmark on shared block storage found SATA/IDE the worst performers, while VirtIO SCSI delivered a 157% IOPS increase and up to 67% lower high-queue-depth latency versus VMware PVSCSI (itself already ahead of SATA/IDE).
- An older qemu-kvm sequential I/O benchmark found VirtIO beating SCSI emulation on reads, but SCSI emulation actually edging out VirtIO on writes in that specific test — "VirtIO always wins" isn't universal.
- A more recent Harvester/Longhorn benchmark found the opposite in several tests — SATA-bus VMs beat VirtIO VMs on write IOPS and sequential read throughput. Storage backend and caching can matter as much as bus type.

**The more reliable signal is CPU cost per I/O operation**, not raw throughput — VirtIO's paravirtualized path genuinely does less host-CPU work per operation, even when raw MB/s numbers land close to SATA's. This matters most when many VMs share host cores, the workload is IOPS-heavy (databases, small random I/O, high queue depth), or the backing storage is fast (NVMe/SSD, where CPU overhead — not the disk — becomes the bottleneck).

| Scenario | Expected gap |
|---|---|
| Light desktop use, sequential I/O, HDD-backed | Small to negligible |
| Moderate everyday VM usage, SSD-backed | Noticeable but modest, usually favoring VirtIO/SCSI |
| Heavy random I/O, high queue depth, NVMe-backed, or many VMs sharing a host | Large gap in VirtIO/SCSI's favor — sometimes 1.5–2.5x+ |

**Bottom line:** I default to VirtIO SCSI (Single, with IOThread) since it costs nothing once drivers are installed, but I don't expect a fixed guaranteed percentage improvement — benchmark the actual workload with `fio` if it matters.

---

## 5. The IOThread option

By default, QEMU funnels most VM work — CPU emulation, device emulation, disk I/O — through a small number of threads, often one shared main event-loop thread. A burst of disk activity competing with everything else on that thread can stall unrelated VM work.

**With IOThread enabled**, QEMU creates a dedicated OS thread per storage *controller* rather than funneling everything through one shared thread, so disk I/O can run truly in parallel with other VM activity on a different host core. This only pays off with **VirtIO SCSI Single** specifically, since IOThreads work at the controller level: plain VirtIO SCSI puts every disk on one shared controller (one thread regardless of disk count), while VirtIO SCSI Single gives each disk its own controller and thus its own independent IOThread — the benefit multiplies as disks are added.

---

## 6. Disk image formats: raw vs. qcow2

Separate from bus/controller — this is the file format storing the disk's actual bytes on the host filesystem.

**raw** — bytes stored exactly as-is, no container format. Best raw performance, trivially inspectable with `dd`/`losetup`, can still be sparse on a supporting filesystem. No built-in snapshots/compression/encryption/backing-files; non-sparse files consume their full size immediately. Best when maximum I/O performance matters, or the storage layer (ZFS/LVM-thin/Ceph) already provides snapshots/thin-provisioning.

**qcow2** — QEMU's native container format with internal metadata. Thin-provisioned by default, internal point-in-time snapshots, backing-file/CoW chains for templating, optional compression/encryption (LUKS, qcow2 v3+). More per-I/O overhead, generally slower than raw for random I/O. Best for VM-level snapshots or plain-filesystem storage (ext4/XFS) without native thin-provisioning.

| | raw | qcow2 |
|---|---|---|
| Performance | Best | Good, with overhead |
| Space usage | Sparse if filesystem supports it | Sparse/thin-provisioned by design |
| Snapshots | None built-in | Yes (internal) |
| Backing files / CoW | No | Yes |
| Compression / Encryption | No | Yes (optional) |
| Portability | Directly usable with `dd`/loop mounts | Needs `qemu-img` tooling |

> **Proxmox note:** on ZFS/Ceph/LVM-thin storage, Proxmox defaults to **raw**, since those backends already provide snapshots/thin-provisioning. On plain directory-based storage, it defaults to **qcow2** for the same features.

> **Critical `dd` caveat:** since raw is a direct byte-for-byte image, `dd` can move data between a raw file and a physical block device in either direction (Section 11). A **qcow2 file cannot be `dd`'d directly** — its bytes are wrapped in QEMU's container format. Convert first:
> ```bash
> qemu-img convert -O raw disk.qcow2 disk.raw
> ```

---

## 7. Discard / TRIM

**The problem:** when I delete files inside a VM, the guest marks that space free, but the host-side disk file (e.g. a `.qcow2`) doesn't automatically shrink, since the host has no idea those blocks were freed.

**The fix — Discard mode `unmap`:**
```
[Guest OS deletes file] ──> [Sends TRIM command] ──> [QEMU unmap] ──> [Host reclaims space]
```
When the guest issues TRIM, `unmap` passes it through to the host, which punches holes in the backing file (or issues real TRIM to the underlying SSD). It isn't the default because reclaiming space can cause temporary latency spikes, and multiple VMs aggressively reclaiming/reallocating space on a nearly-full host drive could exhaust it and crash every VM at once.

**Version requirements** (this trips people up more than it looks):

| Component | Requirement |
|---|---|
| virtio-scsi discard (QEMU) | QEMU ≥ 1.5 — works on both `i440fx` and `q35` |
| virtio-blk discard (QEMU) | QEMU ≥ 4.0 — **but only on `q35`.** Silently doesn't work on `i440fx` regardless of QEMU version |
| Linux guest kernel (virtio-blk) | ≥ 4.20 (virtio-scsi discard has worked much longer) |
| Windows viostor/vioscsi driver | Any `virtio-win` build after ~June 2019 |
| SATA/AHCI (ATA TRIM) | Stable since roughly QEMU 1.1–1.4; needs `discard=unmap` on the drive |

**Practical takeaway:** if I'm on VirtIO Block and TRIM matters, I double-check the VM is on `q35` — an `i440fx` VM silently fails to reclaim space no matter how new QEMU is. VirtIO SCSI sidesteps this entirely, one more reason it's the safer default.

**Windows caveat:** even with modern drivers, running "Optimize" on a VirtIO SCSI disk has reportedly caused excessive CPU load and *rewritten* data instead of cleanly trimming it in some cases — worth testing the actual setup rather than assuming drivers alone guarantee efficient reclamation.

**Which bus/controller supports discard:** VirtIO SCSI/SCSI Single (reliable, longest track record) > SATA (works with modern QEMU + `discard=unmap`) > VirtIO Block (works, but `q35`-only) > IDE (essentially unsupported) > LSI/MegaRAID (not designed for this).

**Triggering it:** Windows — *Defragment and Optimize Drives* → **Optimize** (or `schtasks`/Task Scheduler running `defrag /O C:` weekly). Linux — `sudo fstrim -av` manually, or `sudo systemctl enable --now fstrim.timer` for weekly automation.

> ⚠️ I avoid continuous trimming (the `-o discard` mount flag on Linux) — it causes constant micro-stutters. Scheduled weekly TRIM is the better trade-off.

---

## 8. Verifying discard/TRIM is actually working

Two layers to check: whether the **host** is actually passing discard through, and whether the **guest** sees and uses it.

**Host side:**
- Virt-manager: VM details → disk → **Discard mode** should read `unmap`, not `ignore`/`no`.
- CLI: `virsh dumpxml <vmname> | grep -A3 "<disk"` — look for `discard='unmap'`. If using VirtIO Block, also confirm the machine type is `q35`.

**Linux guest:**
1. `lsblk -D` — check `DISC-GRAN`/`DISC-MAX`; `0` means no discard support.
2. `cat /sys/block/vda/queue/discard_max_bytes` (swap `vda` for `sda` on SATA/SCSI) — non-zero means the block layer sees support.
3. `sudo fstrim -v /` — the most definitive test, confirms the whole chain end-to-end.
4. `uname -r` — needs ≥ 4.20 for virtio-blk discard specifically.

**Windows guest:**
1. `fsutil behavior query DisableDeleteNotify` — `0` means TRIM is allowed at the OS level (`fsutil behavior set disabledeletenotify NTFS 0` to fix).
2. **Settings → System → Storage → Optimize Drives** (`dfrgui.exe`) — "Media type" column: **Thin provisioning drive** means Windows sees it as UNMAP-capable; **Hard disk drive** means Optimize will do nothing regardless of host config (double-check the `virtio-win` driver is a mid-2019+ build if this shows unexpectedly).
3. Run Optimize, then cross-check with `qemu-img info /path/to/vm.qcow2` before/after — if the disk-size line shrinks, the chain is confirmed working.

---

## 9. Cache modes

Cache mode controls how data travels between the guest, the host's RAM (page cache), and the physical drive — a direct speed-vs-safety trade-off.

| Cache Mode | Host RAM Cache Used? | Speed | Host Crash Safety | Best Used For |
|---|---|---|---|---|
| **`writeback`** *(default)* | Yes | Very fast | Medium | General desktop VMs, daily use, testing |
| **`none`** | No | Fast (on SSDs) | High | Production servers, databases, fast NVMe SSDs |
| **`writethrough`** | Reads only | Slow | High | Critical legacy systems where write speed doesn't matter |
| **`unsafe`** | Yes | Maximum | None | Brand-new OS installs, throwaway builds |

`writeback` reports a write as done the instant it hits host RAM — snappy, but an unexpected host power loss can lose whatever was in that cache and corrupt the VM's filesystem. `none` bypasses the host RAM cache entirely — on a modern SSD/NVMe the speed difference versus `writeback` is small, but data is far safer from a sudden outage, and it avoids double-caching (data cached in both guest and host RAM at once).

---

## 10. Reclaiming space from the host

If a `.qcow2` file has grown bloated from deleted-but-never-trimmed files, I can shrink it directly from the host. The VM **must** be fully shut down first, or the disk can be corrupted.

**Method A — `virt-sparsify` (preferred):**
```bash
sudo apt install libguestfs-tools
sudo virt-sparsify --in-place /path/to/your-vm.qcow2
```

**Method B — `qemu-img` (no extra tools):**
```bash
qemu-img convert -c -O qcow2 original.qcow2 shrunken.qcow2
mv original.qcow2 backup.qcow2
mv shrunken.qcow2 original.qcow2
```

To verify actual physical footprint (not the virtual/max size a file manager usually shows):
```bash
qemu-img info /path/to/your-vm.qcow2
```

---

## 11. Physical ⇄ virtual migration (P2V / V2P)

Since a **raw** image is a byte-for-byte copy of a block device — partition table, bootloader, everything — `dd` (or `qemu-img convert`) can move data between a raw file and a physical disk in either direction. The real challenge in both directions isn't moving bytes, it's making sure the guest OS has a working, **boot-start** driver for whatever storage controller it wakes up on.

**Moving the bytes:**
```bash
# Physical disk → raw file (P2V)
sudo dd if=/dev/sdX of=/path/to/disk.raw bs=4M status=progress conv=fsync,noerror

# raw file → physical disk (V2P)
sudo dd if=/path/to/disk.raw of=/dev/sdX bs=4M status=progress conv=fsync

# qcow2 must be converted to raw first — not directly dd-able
qemu-img convert -O raw disk.qcow2 disk.raw
```
Caveats: target the whole disk (`/dev/sdX`), not a partition; make sure the source is quiescent (VM shut down / disk unmounted) before imaging; resize the partition/filesystem afterward if the destination size differs.

**P2V (Windows):** attach the imaged disk as **SATA** first (Windows' inbox `storahci.sys` AHCI driver is boot-start by default on any stock install) so it boots cleanly; install the `virtio-win` driver package while still on SATA, staging it into the driver store; shut down, switch to **VirtIO SCSI Single**, reboot. (`virt-v2v` can inject the driver offline and skip the SATA bootstrap step.)

**P2V (Linux):** attach as SATA first; regenerate the initramfs to make sure `virtio_blk`/`virtio_scsi` are included (some hostonly builds only include what was detected at build time):
```bash
# Debian/Ubuntu
echo -e "virtio_blk\nvirtio_scsi" >> /etc/initramfs-tools/modules
update-initramfs -u
# RHEL/Fedora (dracut)
dracut --add-drivers "virtio_blk virtio_scsi" --force
```
Then switch to VirtIO SCSI Single and reboot.

**V2P (Windows):** the same risk in reverse — on a stock install, only the driver for the controller actually used at install time is boot-start; e.g. `storahci` (SATA) is `BOOT_START` while `stornvme` (NVMe) sits at `DEMAND_START` even if NVMe was never used. Before converting: flip the destination driver to boot-start while still bootable under the old config (`sc config stornvme start= boot` for a SATA→NVMe move; the same idea for VirtIO→standard hardware). If already imaged and unbootable, boot from install media/WinPE, load the offline `SYSTEM` hive, and change the relevant service's `Start` value from `3` to `0`. If the target has real hardware RAID (LSI MegaRAID/PERC/Smart Array), that's a genuine third-party driver — inject it into the offline image via DISM `/Add-Driver`, same idea as staging `virtio-win` in the P2V direction.

**V2P (Linux):** check whether the initramfs includes `ahci`/`nvme` modules — a hostonly rebuild after switching to VirtIO may only contain virtio modules:
```bash
# dracut
dracut --no-hostonly --regenerate-all
# Debian/Ubuntu
echo -e "ahci\nnvme" >> /etc/initramfs-tools/modules
update-initramfs -u
```
Generally more forgiving than Windows, since most distro installers build a broad initramfs by default.

**General principle for both directions:** boot successfully on the *current* controller, stage/enable the *destination* controller's driver while still in a working state, then switch and reboot — never commit to the switch blind. Dedicated backup/restore tools (Macrium Reflect, Acronis, Clonezilla) automate this driver staging and are worth using over manual registry surgery for anything beyond a one-off.

---

## 12. Summary checklist for a new VM

**Bus/controller:** default to VirtIO SCSI Single + IOThread for both Linux/Windows (install `virtio-win` for Windows); SATA only as a legacy fallback or temporary bootstrap bus; avoid IDE except for genuinely ancient guests or the CD-ROM slot; avoid LSI/MegaRAID for new Windows VMs (fine on Linux, but VirtIO SCSI is faster regardless); don't assume a fixed SATA-vs-VirtIO gap — benchmark with `fio` if it matters.

**Disk format:** raw for max performance or when the storage backend already provides snapshots/thin-provisioning; qcow2 for QEMU-level snapshots or plain-filesystem storage without built-in thin-provisioning.

**Discard/cache on SSD/NVMe hosts:** Discard `unmap`, Cache `none`, VirtIO SCSI (or SCSI Single) for reliable discard (confirm `q35` if stuck on plain VirtIO Block); schedule TRIM weekly rather than continuously; verify end-to-end per Section 8 rather than assuming it works.

**Discard/cache on spinning-HDD hosts:** keep Cache at `writeback` for performance, but put the host on a UPS to protect against power-loss data corruption.

**Migration (P2V/V2P):** always convert qcow2 → raw before `dd`; always stage the destination controller's driver as boot-start before switching, in either direction.
