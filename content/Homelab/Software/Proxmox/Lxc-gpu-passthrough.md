---
title: LXC GPU passthrough
description: Passing an NVIDIA GPU through to a Proxmox LXC container, and sharing NAS storage into it over CIFS.
tags:
  - Homelab
  - Proxmox
---
# Why an LXC instead of a full VM

An [[Homelab/Software/Proxmox/Proxmox-glossary#LXC|LXC]] container shares the Proxmox host's kernel instead of virtualizing its own, which makes GPU passthrough dramatically simpler than full PCI passthrough to a VM — I just need to bind-mount the right device nodes into the container rather than dealing with IOMMU groups and a dedicated PCI device. This is reference material for a container I might stand up on this Proxmox host in the future; I haven't put a specific service behind it yet.

# NVIDIA driver setup on the Proxmox host

The driver has to be installed on the Proxmox host itself, not just inside the container, since the container shares the host's kernel and driver:

1. Update the host to the latest version, and reboot if the kernel was updated.
2. Install the driver build packages: `apt install build-essential pve-headers-$(uname -r)`.
3. Blacklist the open-source `nouveau` driver so it doesn't conflict with NVIDIA's proprietary one — edit `/etc/modprobe.d/blacklist-nouveau.conf`:

```
blacklist nouveau
options nouveau modeset=0
```

4. Regenerate the initramfs: `update-initramfs -u`.
5. Unload `nouveau` immediately (rather than waiting for a reboot) via `rmmod nouveau`, and confirm it's gone with `lsmod | grep nouveau` returning nothing.
6. Find the matching NVIDIA server driver version: check which `nvidia-utils-server` versions [Ubuntu packages](https://packages.ubuntu.com/search?suite=questing&section=all&arch=any&keywords=nvidia-utils+server&searchon=names), then find the corresponding installer on [NVIDIA's driver download page](https://www.nvidia.com/en-us/drivers/).
7. Download and run the installer on the Proxmox host: `wget <driver URL>`, `chmod +x <filename>`, then `./<filename>`, accepting the defaults through its menu.
8. Verify with `nvidia-smi` on the host.

# Passing the GPU into the container

1. Create the LXC container (I used Ubuntu), update it, and shut it down before editing its config.
2. Edit `/etc/pve/nodes/<node-name>/lxc/<container-id>.conf` and add:

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

3. Start the container and install the *same* NVIDIA server driver version inside it that was installed on the host (drivers must match between host and container since they share a kernel). Verify with `nvidia-smi` inside the container too.

# Gotchas

- **Docker inside the LXC needs `fuse` and `nesting` enabled** (Proxmox's container Options → Features tab) to run without duplicating files and wasting storage.
- **A Docker container inside the LXC using the GPU needs the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#with-apt-ubuntu-debian)** installed inside the LXC as well.
- If the NVIDIA container CLI reports it can't find any device filters attached to the cgroup, cgroups need to be disabled for it: stop all running containers, uncomment and set `no-cgroups = true` in `/etc/nvidia-container-runtime/config.toml`, then regenerate the CDI config with `nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml`.

# Mounting NAS storage into an LXC over CIFS

Separately from GPU passthrough, I can mount an [[Homelab/Software/OpenMediaVault/Shares|OpenMediaVault share]] into an LXC over CIFS from the Proxmox host side:

1. Inside the LXC, as a non-root user, create a group that will own the share: `groupadd -g 10000 lxc_shares`, then add the target user to it: `usermod -aG lxc_shares <username>`.
2. Shut the LXC down.
3. On the Proxmox host, create a mount point: `mkdir -p /mnt/TrueNAS/<share>`.
4. Add the CIFS mount to the host's `/etc/fstab`:

```
//<NAS-IP>/<share>/ /mnt/TrueNAS/<share> cifs _netdev,x-systemd.automount,noatime,uid=100000,gid=110000,dir_mode=0770,file_mode=0770,user=<username>,pass=<password> 0 0
```

5. Bind-mount it into the container's config (`/etc/pve/lxc/<container-id>.conf`):

```
mp0: /mnt/TrueNAS/<share>,mp=/mnt/<share>
```

**Flagged concern:** the `fstab` line above embeds the CIFS credentials (`user=`/`pass=`) in plaintext directly in `/etc/fstab`, which is world-readable by default on most systems. I haven't actually deployed this yet, so I've flagged it in `Migration Q&A.json` rather than silently normalizing or silently leaving it — a `credentials=/path/to/file` reference (readable only by root) is the safer standard approach if this gets used for real.
