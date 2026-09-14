---
title: Setting Up OpenMediaVault as a NAS
description: Installing OpenMediaVault in a Proxmox VM on the laptop, picking a filesystem, and sharing a drive over NFS, including the permission/ACL gotcha that blocks access by default.
tags:
  - Asus-X555q
  - OpenMediaVault
  - Networking
---
## Why OpenMediaVault

After [[Failed-laptop/Asus-X555q/Proxmox-installation|installing Proxmox]] on the laptop, my next step was turning it into a [[Asus-x555q-glossary#NAS|NAS]]. I ruled out [TrueNAS](https://www.truenas.com/) — its documented minimum requirements (2 cores, 8 GB RAM, a 16 GB boot device, and two identically-sized storage devices) would consume most of the laptop's resources on their own:

![[truenas-scale-minimum-hardware-requirements.png]]
*TrueNAS Scale's published minimum hardware requirements — the constraint that ruled it out for this hardware.*

I chose [[Asus-x555q-glossary#OpenMediaVault|OpenMediaVault]] (OMV) instead: a lighter Debian-based NAS OS, versus manually assembling the same functionality from a bare Linux distro.

## VM layout

I created the OMV VM with 2 CPU cores, 4 GB RAM, the Ethernet NIC connected to the switch, and two virtual disks: a 10 GB disk for the OMV OS itself, and a 500 GB disk dedicated to NAS storage. Most NAS software expects the OS to live on its own disk, separate from the disk(s) it manages for shares.

![[omv-vm-two-disk-layout-diagram.png]]
*The VM's two-disk layout: a small OS disk and a large storage disk kept separate.*

## Installation and network gotchas

I accidentally left the installer language on Russian; I carried the install through anyway and it completed successfully, then I repeated it with the correct language for day-to-day use — a full reinstall from the ISO takes only a few minutes and avoids working through menus in an unfamiliar language. The install crashed the underlying Proxmox host with a kernel panic partway through; a manual reboot brought Proxmox back and I only needed to reinstall the OMV VM.

I configured the network with a static IP (no gateway or DNS, since nothing on this private switch has an internet route), following the same "no static IPs on the real network, so keep this network private and self-contained" reasoning as the [[Failed-laptop/Asus-X555q/Proxmox-installation#NAT, bridging, and routing for VM traffic|Proxmox networking setup]].

After install, the web UI was unreachable over both HTTP and HTTPS despite the VM responding to ping. The cause: the browser (I tested both LibreWolf and Chromium) was silently rewriting `http://` requests to `https://` without any visible redirect, and OMV's UI wasn't listening on HTTPS.

![[librewolf-http-silently-upgraded-to-https-connection-refused.png]]
*The browser refusing to connect after silently upgrading an http:// request to https:// — the actual bug was the browser's HTTPS-upgrade behavior, not the OMV install.*

Forcing the browser to keep the `http://` scheme (re-entering the URL until it stuck) resolved it, and the default `admin` / `openmediavault` login worked from there. I changed the default credentials shortly after install (I didn't document the replacement at the time).

## Choosing a filesystem for the storage disk

OMV doesn't support [ZFS](https://docs.openmediavault.org/en/latest/administration/storage/filesystems.html). Of the supported options (BTRFS, EXT4, F2FS, JFS, XFS), I chose [[Asus-x555q-glossary#BTRFS|BTRFS]] for its built-in snapshots, self-healing, and compression — features EXT4 deliberately omits in favor of simplicity. I formatted the 500 GB disk BTRFS in single-device mode and it mounted without issue.

## Shared folder and NFS share

I created a shared folder (`NAS`, at the root of the BTRFS filesystem) with the predefined permission set `Administrator: read/write, Users: read/write, Others: no access`, then exposed it over [[Asus-x555q-glossary#NFS|NFS]] rather than [[Asus-x555q-glossary#SMB/CIFS|SMB/CIFS]] — my client device is Linux, and NFS's Unix-native permission model was a better fit than reusing the Windows-oriented SMB setup I'd used previously. I configured the client's own machine with the `nfs-utils` package and enabled the NFS client service.

NFS is **disabled by default** in OMV even after a share is configured — the share won't be reachable until you switch on the NFS service itself under `Services → NFS → Settings`.

![[omv-nfs-service-disabled-by-default.png]]
*NFS shown disabled by default in OMV's service settings — easy to miss after only configuring the share itself.*

With the service enabled, the share mounted successfully from the client with `mount -t nfs <omv-ip>:/ /mnt/OMV`:

![[omv-nfs-share-mounted-with-restrictive-permission-bits.png]]
*The mounted NFS export listed from the client — reachable, but with `drwxrws---` permission bits owned by root, which blocks write access (see below).*
### Permission denied, even as root

Even after granting the client's user account read/write on the shared folder, and even when I `su`-ed to root on the client, I couldn't enter the folder:

![[nfs-share-permission-denied-even-as-root.png]]
*`cd` into the mounted share refused for both a normal user and root — a sign the problem was server-side ownership, not client permissions.*

The actual cause was the shared folder's [[Asus-x555q-glossary#ACL|ACL]]: OMV's regular "Permissions" tab only sets the classic owner/group/other mode bits and doesn't touch the ACL, and the folder's ACL owner was still `root`.

![[omv-shared-folder-acl-owner-set-to-root.png]]
*The shared folder's access control list, still showing `root` as the owning user — the actual blocker, independent of the read/write permission already granted.*

Switching the ACL's owning user to my client's own user account fixed it — the folder became writable and I could create a test file:

![[nfs-share-writable-after-acl-owner-fix.png]]
*Read/write access confirmed after correcting the ACL owner, including successfully creating a test file.*

After the fix, I redid the NFS mount to point directly at the share's export path (`<omv-ip>:/export/NAS`) instead of OMV's top-level `/export`, to avoid inheriting less predictable permission behavior from the parent NFSv4 pseudo-filesystem.

## Gotchas

- If an OMV share is unreachable after being fully configured, check whether the NFS/SMB *service* itself is enabled — configuring a share does not enable the underlying service.
- If a share is unwritable even for root, check the shared folder's ACL owner in OMV, not just its Unix permission bits — OMV's basic permission editor and its ACL editor are separate, and only the ACL editor changes file ownership.
- Prefer mounting a share's specific export path over OMV's parent `/export` pseudo-filesystem root, to avoid inherited permission surprises.
