---
title: "Pt 5, Setting up Looking Glass"
description: Using Looking Glass and a virtual display driver to control the VM without needing a physical monitor on the passthrough GPU.
tags:
  - Virtual_Machines
  - Looking_Glass
---
# Pt 5, Setting up Looking Glass

# Prerequisites
- [[Pt 1, Creating-the-vm|Pt 1, Creating the VM]]
- [[Pt 2, Enabling-iommu|Pt 2, Enabling IOMMU]]
- [[Pt 3, Isolating-the-gpu|Pt 3, Isolating the GPU]]
- [[Pt 4, Attaching-the-gpu|Pt 4, Attaching the GPU to the VM]]

---

## Why

Requiring a monitor to be physically plugged into the passthrough GPU at all times is inconvenient on a laptop. [[Windows-gaming-vm-glossary#Looking Glass|Looking Glass]] reads the GPU's raw frames directly so the VM can be viewed/controlled without a monitor on that GPU — but it needs [[Windows-gaming-vm-glossary#IVSHMEM|IVSHMEM]] set up first.

## Calculating the IVSHMEM size

1. Multiply the desired max resolution's width and height.
2. Multiply by the BPP (4 for SDR, 8 for HDR).
3. Multiply by 2 to get the frame size in bytes.
4. Divide by 1,048,576 to convert to MiB.
5. Add 10.
6. Round up to the nearest power of two.

Example for 1920×1200 SDR: `1920 × 1200 = 2,304,000` → `× 4 = 9,216,000` → `× 2 = 18,432,000` → `÷ 1,048,576 ≈ 17.58` → `+ 10 ≈ 27.58` → nearest power of two ≥ 27 is **32 MiB**.

## Adding IVSHMEM to the VM

Enable XML editing in Virt Manager (**Edit → Preferences → General → Enable XML editing**), then add before `</devices>` in the VM's XML:

```xml
<shmem name='looking-glass'>
  <model type='ivshmem-plain'/>
  <size unit='M'>32</size>
</shmem>
```

Create `/etc/tmpfiles.d/10-looking-glass.conf` (replace `user` with the actual username) so the shared-memory file exists on boot:

```
#  Path                    Mode  UID   GID  Age  Argument
f  /dev/shm/looking-glass  0660  user  kvm  -    -
```

Apply it immediately with `systemd-tmpfiles --create /etc/tmpfiles.d/10-looking-glass.conf`.

## Installing Looking Glass

- Host: install the [Looking Glass Client](https://looking-glass.io/docs/B7/install_client/) — on Arch-based distros, the [AUR package](https://aur.archlinux.org/packages/looking-glass) builds it automatically.
- Guest: install the Windows Host Binary from the Looking Glass download page, making sure the **IVSHMEM Driver** component is checked.

With the GPU's video hardware set to `None` in Virt Manager (so the GPU is the only display) and the VM started, launch the Looking Glass client on the host — it exits immediately if there's no SPICE server to connect to, so give Windows and the Looking Glass server time to start first.

![[looking-glass-client-displaying-vm.png]]
*The Looking Glass client successfully connected and rendering the VM's display — confirms the IVSHMEM + Looking Glass pipeline is working end-to-end.*

If the display stays blank with CPU usage pinned at a constant value, try disabling **ROM BAR** for the GPU device in the VM's PCI host device settings:

![[cpu-usage-flatlined-black-screen-issue.png]]
*CPU usage flat-lined at a fixed percentage with no display output — the symptom that indicated ROM BAR needed to be disabled for this GPU.*

## Removing the physical-monitor requirement

The GPU still needs *something* to think is plugged in. Two options: a physical HDMI/DP dummy plug, or a virtual display driver. I used the [Virtual Display Driver](https://github.com/VirtualDrivers/Virtual-Display-Driver) project for this: install the latest release inside the guest, run `VDD Control.exe`, click **Install Driver** (confirm the driver-install prompt). Once the virtual display shows up under **Device Manager → Display Adapters**, the physical monitor can be unplugged and Looking Glass keeps working.

Custom resolutions (e.g. for a 16:10 panel, where the driver only ships 16:9 presets) can be added to `C:\VirtualDisplayDriver\vdd_settings.xml`:

```xml
<resolution>
  <width>1920</width>
  <height>1200</height>
  <refresh_rate>30</refresh_rate>
</resolution>
```

The refresh rate value in the resolution entry doesn't matter much — a separate setting in the same file applies the real refresh rate to all resolutions, once the driver is restarted from the Virtual Driver Control GUI.

---

Next: [[Pt 6, Dynamic-driver-switching|Pt 6, Dynamic driver switching]].
