---
title: Auto-Mounting the NAS Share with autofs
description: Automatically mounting the OpenMediaVault NFS share on demand from a laptop client, instead of a fixed fstab mount that would slow every boot.
tags:
  - Asus-X555q
  - OpenMediaVault
  - Networking
---

## Overview

This page covers auto-mounting the OpenMediaVault NFS share on demand from a laptop client, instead of a fixed fstab mount that would slow every boot.

## Why Not /etc/fstab

Adding the NFS share to `/etc/fstab` on the client would mount it unconditionally at boot. That's fine for a machine that's always on the same network as the NAS, but the client here is a laptop that also boots away from home (e.g., in public places) — a boot-time NFS mount would stall startup waiting for a host that isn't reachable.

## Using autofs

autofs mounts a share only when something actually accesses it, and unmounts it again after a configurable idle timeout. This eliminates manual mount/unmount and boot-time dependency on the NAS being reachable.

### Distribution Differences

The autofs master map file's location differs by distribution:

- **Debian, Ubuntu, Fedora, RHEL-based distros:** `/etc/auto.master`
- **Arch Linux and Arch-based distros:** `/etc/autofs/autofs.master`

### Configuration

Once located, one line was added to the master map:

```
/mnt /etc/autofs/auto.nfs --timeout=60
```

This tells autofs to automount shares under `/mnt`, using the map file `/etc/autofs/auto.nfs` to decide what to mount there, and to unmount an idle share after 60 seconds. The map file itself doesn't exist by default, so it was created with one line per share:

```
OMV -rw,soft,rsize=8192,wsize=8192 192.168.100.4:/export/NAS
```

The three parts are: the local mount point name under `/mnt` (`OMV`), the NFS mount options, and the share's NFS export path. After editing either file, the autofs service needs a restart to pick up the change.

Accessing `/mnt/OMV` (e.g., with `ls`) triggers the mount on demand; after the configured timeout with no activity, it unmounts automatically.

See [autofs verification screenshot](Images/autofs-nfs-share-automount-and-idle-unmount-verification.png) for the share mounting on first access and disappearing again after the idle timeout — confirming autofs is managing the mount instead of a static `fstab` entry.

## Gotcha: Other Automounts Under the Same Parent Directory

Once autofs owns `/mnt`, anything else normally mounted directly under `/mnt` (in this case an NTFS Windows drive at `/mnt/Windows`) gets hidden, since autofs manages that whole directory.

### Resolution

The other mount was added to the *same* map file family rather than a separate untracked one — autofs only supports one map file per parent directory declared in the master map:

```
Windows -fstype=ntfs3,rw,uid=1000,gid=1000 :/dev/disk/by-uuid/A890A4F690A4CC5E
```

The `--ghost` option was also added to the master map line so autofs pre-creates the empty mount-point directories (`/mnt/OMV`, `/mnt/Windows`) instead of only creating them on first access.

## Gotchas

- **NFS service enabled** — If a share isn't mounting through autofs, check that the NFS service is actually enabled on the server side first — a disabled service on OMV looks identical to a client-side autofs misconfiguration
- **Other mounts under autofs-managed parent** — Any other mount points expected under the same parent directory autofs manages (e.g., `/mnt`) need to be added to autofs's own map, not configured independently — autofs owns that directory once it's in control of it
