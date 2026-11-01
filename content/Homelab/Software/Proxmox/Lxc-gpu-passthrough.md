---
title: Proxmox LXC GPU Passthrough
description: Passing an NVIDIA GPU through to a Proxmox LXC container and sharing NAS storage into it over CIFS.
tags:
  - Homelab
  - Proxmox
---

## Overview

This page covers passing an NVIDIA GPU through to a Proxmox LXC container and sharing NAS storage into it over CIFS.

## Why an LXC Instead of a Full VM

An LXC container shares the Proxmox host's kernel instead of virtualizing its own, which makes GPU passthrough dramatically simpler than full PCI passthrough to a VM. Instead of dealing with IOMMU groups and a dedicated PCI device, the right device nodes are bind-mounted into the container.

This is reference material for a container that might be stood up on this Proxmox host in the future; no specific service has been put behind it yet.

## NVIDIA Driver Setup on the Proxmox Host

The driver has to be installed on the Proxmox host itself, not just inside the container, since the container shares the host's kernel and driver.

### Installation Steps

1. **Update the host** — Update to the latest version and reboot if the kernel was updated
2. **Install driver build packages:** `apt install build-essential pve-headers-$(uname -r)`
3. **Blacklist the open-source `nouveau` driver** — Edit `/etc/modprobe.d/blacklist-nouveau.conf`:

   ```
   blacklist nouveau
   options nouveau modeset=0
   ```

4. **Regenerate initramfs:** `update-initramfs -u`
5. **Unload `nouveau` immediately** — Via `rmmod nouveau`, and confirm it's gone with `lsmod | grep nouveau` returning nothing
6. **Find the matching NVIDIA server driver version** — Check which `nvidia-utils-server` versions Ubuntu packages have, then find the corresponding installer on NVIDIA's driver download page
7. **Download and run the installer on the Proxmox host:** `wget <driver URL>`, `chmod +x <filename>`, then `./<filename>`, accepting the defaults through its menu
8. **Verify with `nvidia-smi` on the host**

## Passing the GPU into the Container

1. **Create the LXC container** — Used Ubuntu, update it, and shut it down before editing its config
2. **Edit `/etc/pve/nodes/<node-name>/lxc/<container-id>.conf`** and add:

   ```
   lxc.cgroup2.devices.allow: c 195:* rwm
   lxc.cgroup2.devices.allow: c 243:* rwm
   lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file
   lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
   lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
   lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
   lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
   ```

   If the host has more than one GPU, `/dev/nvidia0` needs to point at the right index — `nvidia-smi` on the host lists each GPU's ID on the left of its name.

3. **Start the container and install the same NVIDIA server driver version inside it** that was installed on the host (drivers must match between host and container since they share a kernel). Verify with `nvidia-smi` inside the container too.

## Gotchas

- **Docker inside the LXC needs `fuse` and `nesting` enabled** (Proxmox's container Options → Features tab) to run without duplicating files and wasting storage
- **A Docker container inside the LXC using the GPU needs the NVIDIA Container Toolkit installed inside the LXC as well**
- **Cgroup device filters** — If the NVIDIA container CLI reports it can't find any device filters attached to the cgroup, cgroups need to be disabled for it: stop all running containers, uncomment and set `no-cgroups = true` in `/etc/nvidia-container-runtime/config.toml`, then regenerate the CDI config with `nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml`

## Mounting NAS Storage into an LXC over CIFS

Separately from GPU passthrough, an OpenMediaVault share can be mounted into an LXC over CIFS from the Proxmox host side.

### Mount Steps

1. **Inside the LXC, as a non-root user, create a group** that will own the share: `groupadd -g 10000 lxc_shares`, then add the target user to it: `usermod -aG lxc_shares <username>`
2. **Shut the LXC down**
3. **On the Proxmox host, create a mount point:** `mkdir -p /mnt/TrueNAS/<share>`
4. **Add the CIFS mount to the host's `/etc/fstab`:**

   ```
   //<NAS-IP>/<share>/ /mnt/TrueNAS/<share> cifs _netdev,x-systemd.automount,noatime,uid=10000,gid=110000,dir_mode=0770,file_mode=0770,user=<username>,pass=<password> 0 0
   ```

5. **Bind-mount it into the container's config (`/etc/pve/lxc/<container-id>.conf`):**

   ```
   mp0: /mnt/TrueNAS/<share>,mp=/mnt/<share>
   ```

### Flagged Concern: Plaintext Credentials

The `fstab` line above embeds the CIFS credentials (`user=`/`pass=`) in plaintext directly in `/etc/fstab`, which is world-readable by default on most systems. This hasn't been deployed yet, so it's flagged in `Migration Q&A.json` rather than silently normalized or silently left — a `credentials=/path/to/file` reference (readable only by root) is the safer standard approach if this gets used for real.

## Related Documentation

- [[Homelab/Software/OpenMediaVault/Shares|OpenMediaVault shares]] — For the NAS shares being mounted
