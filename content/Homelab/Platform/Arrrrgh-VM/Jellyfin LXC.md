---
title: Jellyfin LXC Setup on Proxmox
description: Installation and configuration of a Jellyfin media server in a Debian 13 LXC container on Proxmox with GPU passthrough and NFS mounting.
tags:
  - Homelab
  - Proxmox
  - Jellyfin
  - LXC
  - Debian
  - GPU Passthrough
  - NFS
---

## Overview

This guide details the setup of a Jellyfin media server running in a Debian 13 LXC container on Proxmox. The configuration includes GPU passthrough for hardware acceleration, NFS mounting for media library storage, and proper user ID mapping for file permission handling.

---

## Prerequisites

- Proxmox host with Intel integrated GPU (iGPU) and necessary drivers installed.
- Access to the Proxmox shell and web interface.
- An NFS share available at `10.10.0.13:/export/Arrrrgh`.
- Video group access on the Proxmox host: `card0` and `renderD128` devices available.

---

## Step 1: Create Debian 13 LXC Container

Create a new LXC container using Debian 13 (Trixie):

1. In the Proxmox web interface, create a new LXC container.
2. Select Debian 13 (trixie) as the template.
3. Set the initial disk size (e.g., 8 GB).
4. Configure root password or set up SSH key authentication.
5. Enable nesting for systemd support (required for certain services).

### Container Configuration Details

**General Settings:**
- **Node:** `pve`
- **CT ID:** `106`
- **Hostname:** `Jellyfin`
- **Unprivileged container:** Checked (recommended for security)
- **Nesting:** Disabled
- **Add to HA:** Disabled

**Authentication:**
- SSH public key configured for secure access
- Root password set (or SSH key authentication)

---

## Step 2: Configure Network Settings

Configure the container's network interface:

1. Navigate to the Network tab in the Proxmox web interface.
2. Configure the following settings:

**Network Interface (eth0):**
- **Bridge:** `vmbr0` (Standard Linux bridge on host)
- **MAC Address:** Auto-generated
- **VLAN:** None
- **Firewall:** Enabled
- **MTU:** Same as bridge

**IPv4 Configuration:**
- **Type:** Static
- **Address:** `10.10.0.6/28`
- **Gateway:** `10.10.0.14` (Router/gateway IP)

**IPv6 Configuration:**
- **Type:** Static
- **Address/CIDR:** None
- **Gateway:** Empty

---

## Step 3: Configure GPU Passthrough

### Verify Host GPU Drivers

On the Proxmox host, confirm the iGPU and drivers are working:

```bash
ls -l /dev/dri
# Expected output: card0 (or card1) and renderD128
lspci -k | grep -A3 VGA
# Should show "Kernel driver in use: i915"
```

### Pass Through GPU Devices

Identify the video group ID (GID) on the host:

```bash
getent group video render
```

Example output:
```
video:x:44:
render:x:992:
```

Pass through both the card and render devices to the container:

```bash
pct set <CTID> -dev0 /dev/dri/card0,gid=<video_gid>
pct set <CTID> -dev1 /dev/dri/renderD128,gid=<render_gid>
pct start <CTID>
```

Replace `<CTID>`, `<video_gid>`, and `<render_gid>` with actual values.

### Verify GPU Access in Container

Inside the LXC container, confirm the iGPU devices are accessible:

```bash
ls -l /dev/dri/
```

Expected output:
```
total 0
crw-rw---- 1 root video  226,   0 Nov  1 02:03 card0
crw-rw---- 1 root render 226, 128 Nov  1 02:03 renderD128
```

---

## Step 4: Install Graphics Utilities

Install Intel GPU tools and VAINFO for troubleshooting:

```bash
apt install -y vainfo intel-gpu-tools
```

---

## Step 5: Update System

Update and upgrade the system packages:

```bash
apt update && apt upgrade -y
apt purge -y
```

---

## Step 6: Install Jellyfin

### Verify Install Script

Download and verify the Jellyfin installation script:

```bash
apt install curl -y
curl -s https://repo.jellyfin.org/install-debuntu.sh -O && \
curl -s https://repo.jellyfin.org/install-debuntu.sh.sha256sum -O && \
sha256sum -c install-debuntu.sh.sha256sum
```

`install-debuntu.sh: OK` confirms the checksum is correct.

### Run Installation

Make the script executable and run it:

```bash
chmod +x install-debuntu.sh
./install-debuntu.sh
```

Follow the interactive prompts to complete the installation.

### Configure Jellyfin User for GPU Access

Ensure the jellyfin user has access to the GPU devices:

```bash
id -Gn jellyfin
```

If the `render` and `video` groups are missing, add them:

```bash
usermod -aG render,video jellyfin
systemctl restart jellyfin
```

---

## Step 7: Configure NFS Mount

### Create Mount Directory on Host

On the Proxmox host, create the mount directory:

```bash
mkdir /mnt/Arrrrgh
```

### Add to fstab

Edit the host's `/etc/fstab` to mount the NFS share on boot:

```bash
echo "10.10.0.13:/export/Arrrrgh /mnt/Arrrrgh nfs defaults,_netdev,nofail 0 0" >> /etc/fstab
```

Apply the changes:

```bash
systemctl daemon-reload
mount -a
```

---

## Step 8: Bind Mount NFS into Container

Bind mount the host NFS directory into the LXC container:

```bash
pct set <CTID> -mp0 /mnt/Arrrrgh,mp=/mnt/Arrrrgh,backup=0
```

Arguments:
- First argument: host folder (`/mnt/Arrrrgh`)
- Second argument: container folder (`/mnt/Arrrrgh`)
- Third argument: no backup for mount point

---

## Step 9: Configure User ID Mapping

### Map Container Users to Host Users

Edit the container configuration file:

```bash
nano /etc/pve/lxc/<CTID>.conf
```

Add or modify the ID mapping rules:

```
lxc.idmap: u 0 100000 1000
lxc.idmap: g 0 100000 1000
lxc.idmap: u 1000 1000 1
lxc.idmap: g 1000 1000 1
lxc.idmap: u 1001 101001 64535
lxc.idmap: g 1001 101001 64535
```

**Note:** Unprivileged containers shift UIDs by 100000. Therefore, UID 1000 becomes 101000. Double-check the UID of files on the NFS share.

### Allow Root to Map Permissions

On the Proxmox host, allow root to map permissions:

```bash
echo "root:1000:1" >> /etc/subuid
echo "root:1000:1" >> /etc/subgid
```

---

## Step 10: Restart Container

Reboot the container to apply all configuration changes:

```bash
pct reboot <CTID>
```

---

## Step 11: Configure Jellyfin Web Interface

Access the Jellyfin web interface at `http://10.10.0.6:8096` and complete the setup wizard. Configure:

1. Server name
2. Admin username and password
3. Library locations pointing to `/mnt/Arrrrgh`
4. Transcoding settings for GPU acceleration

---

## Verification

After setup, verify the following:

1. Jellyfin starts without errors.
2. GPU passthrough is working (hardware acceleration enabled).
3. NFS mount is accessible and contains media libraries.
4. User permissions allow proper file access.

---

## Troubleshooting

### GPU Not Detected in Container

- Verify the render and video GIDs on the host match those configured in the container.
- Check that the iGPU drivers are properly loaded on the host.
- Confirm the `/dev/dri` devices have correct permissions.

### NFS Mount Issues

- Verify the NFS share is accessible from the host: `mount | grep Arrrrgh`
- Check that the bind mount is correctly applied: `pct config <CTID> | grep mp0`
- Ensure UID/GID mapping is correct for files on the NFS share.

### Jellyfin Transcoding Fails

- Confirm GPU access within the container: `ls -l /dev/dri/`
- Check Jellyfin logs for transcoding errors.
- Verify the Intel GPU drivers are functional.
