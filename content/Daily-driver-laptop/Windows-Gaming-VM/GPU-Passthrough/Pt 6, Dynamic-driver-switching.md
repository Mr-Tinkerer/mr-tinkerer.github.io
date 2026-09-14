---
title: "Pt 6, Dynamic Driver Switching"
description: Unbinding/rebinding the GPU between the host NVIDIA driver and VFIO at runtime, without rebooting.
tags:
  - Virtual_Machines
  - VFIO
  - Bash_Script
---
# Pt 6, Dynamic Driver Switching

# Prerequisites
- [[Pt 1, Creating-the-vm|Pt 1, Creating the VM]]
- [[Pt 2, Enabling-iommu|Pt 2, Enabling IOMMU]]
- [[Pt 3, Isolating-the-gpu|Pt 3, Isolating the GPU]]
- [[Pt 4, Attaching-the-gpu|Pt 4, Attaching the GPU to the VM]]
- [[Pt 5, Setting-up-looking-glass|Pt 5, Setting up Looking Glass]]

---

## The problem

With [[Pt 3, Isolating-the-gpu|Pt 3]] done, the GPU is permanently claimed by VFIO — using it on the host at all requires editing `/etc/mkinitcpio.conf`, rebuilding the initramfs, and rebooting. That's too much friction for something meant to be quick to use. The goal here is to unbind/rebind the GPU's drivers live, with no reboot.

## Manual unbind/rebind, the simple way

The GPU's PCI address as seen in `/sys/bus/pci/devices` is the *long* form (e.g. `0000:01:00.0`), not the short form (`01:00.0`) printed by `lspci` — Virt Manager's hardware tab shows the long form.

```bash
#!/bin/bash
gpu="0000:01:00.0"
aud="0000:01:00.1"
echo $gpu > /sys/bus/pci/devices/$gpu/driver/unbind
echo $aud > /sys/bus/pci/devices/$aud/driver/unbind
```

Rebinding to the normal drivers (`nvidia` / `snd_hda_intel` here — check with `lspci -nnk` for the actual drivers on a given system) is symmetric:

```bash
echo $gpu > /sys/bus/pci/drivers/nvidia/bind
echo $aud > /sys/bus/pci/drivers/snd_hda_intel/bind
```

`nvidia-smi` confirms the GPU is usable again after rebinding.

The [Arch Wiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Binding_vfio-pci_via_device_ID) documents a more robust variant of this, driven by vendor/device ID rather than raw unbind/bind, with `bind_vfio` / `unbind_vfio` functions and an interactive menu. I adopted that version since it's the more maintained approach; it's illustrative of the transition technique, not the final production script (see below for that).

## Why unbinding from VFIO (not from NVIDIA) needed a different trick

Going the other direction — releasing the GPU from the host's NVIDIA driver so VFIO can claim it — is harder, because something is almost always using the GPU. On this laptop at idle, both Steam and [[Gigabyte-gaming-a16-ga6h-glossary#Niri|Niri]] (the window manager itself) hold the GPU.

Niri can be told to ignore the dedicated GPU and only render through the integrated GPU, via its [debug config options](https://github.com/niri-wm/niri/wiki/Configuration:-Debug-Options#render-drm-device):

```json
debug {
	// Nvidia GPU
	ignore-drm-device "/dev/dri/renderD128"

	// Intel IGPU
	render-drm-device "/dev/dri/renderD129"
}
```

(`ls -l /sys/class/drm/renderD*/device/driver` identifies which `renderD*` node belongs to which GPU.) This config only takes effect on a fresh Niri start — restarting Niri live from within itself did *not* release the GPU; only a config applied before Niri's initial start did.

The other blocker was hidden GPU users `nvidia-smi` doesn't show at all: `fuser -v /dev/nvidia*` reveals them. On this laptop, `nvidia-powerd` was one such hidden holder and had to be stopped as a systemd service (`systemctl stop nvidia-powerd`) before the GPU driver could be released.

The remaining holdout was the GPU's audio device, held by [[Windows-gaming-vm-glossary#WirePlumber|WirePlumber]]/PipeWire. Disabling it requires a WirePlumber rule at `~/.config/wireplumber/wireplumber.conf.d/51-disable-hdmi-devices.conf` (the device name comes from the ALSA card list in `pactl list short`):

```json
monitor.alsa.rules = [
  {
    matches = [
      {
        device.name = "alsa_card.pci-0000_01_00.1"
      }
    ]
    actions = {
      update-props = {
        device.disabled = true
      }
    }
  }
]
```

Restarting WirePlumber (`systemctl --user restart wireplumber`) after toggling `device.disabled` applies the change.

## The fix for the "no such device" error

An early version of the unbind/bind approach that used `/sys/bus/pci/drivers_probe` directly to rebind VFIO after removal hit `echo: write error: no such device` on reconnect — VFIO doesn't know it's allowed to reclaim a device that was already removed and re-added. I couldn't find an answer for this online, so I asked Claude for help, not expecting it to work on the first try — but it did. The reliable fix (suggested by Claude) is to set each device's `driver_override` to `vfio-pci` *before* removing/rescanning it, rather than relying on VFIO to pick the device up on its own:

```bash
echo vfio-pci > /sys/bus/pci/devices/$gpu/driver_override
echo vfio-pci > /sys/bus/pci/devices/$aud/driver_override
echo $gpu > /sys/bus/pci/devices/$gpu/driver/unbind
echo $aud > /sys/bus/pci/devices/$aud/driver/unbind
echo $gpu > /sys/bus/pci/drivers_probe
echo $aud > /sys/bus/pci/drivers_probe
```

Going the other direction (back to the host driver), the `driver_override` is cleared instead of set, so the kernel's normal driver matching takes over again:

```bash
echo "" > /sys/bus/pci/devices/$gpu/driver_override
echo "" > /sys/bus/pci/devices/$aud/driver_override
echo $gpu > /sys/bus/pci/devices/$gpu/driver/unbind
echo $aud > /sys/bus/pci/devices/$aud/driver/unbind
echo $gpu > /sys/bus/pci/drivers_probe
echo $aud > /sys/bus/pci/drivers_probe
```

This has never gotten stuck on reconnect, unlike the plain `drivers_probe` approach.

## Running user-session commands from a root script

The full start/stop scripts (see [[Pt 7, Automating-with-libvirt-hooks|Pt 7]]) run as root, but some of their steps (`notify-send`, restarting a `--user` systemd unit) need to run inside the logged-in user's session. This `run_as_user` helper temporarily becomes that user with the right session environment:

```bash
run_as_user() {
  su - "$USER" -c "
    export XDG_RUNTIME_DIR=$XDG_RUNTIME_DIR
    export DBUS_SESSION_BUS_ADDRESS=$DBUS_SESSION_BUS_ADDRESS
    $1
  "
}
```

Claude wrote this helper and the process-listing logic around it for me. I reuse this function as-is (not duplicated) in [[../Gigabyte-Gaming-A16-Ga6h/Battery-management|the laptop's battery-management script]], which needs to run `dms`/`notify-send` commands as the logged-in user from a root-invoked udev rule.

## Final scripts

The final, maintained start/stop scripts (which combine driver switching, GPU-process notification, and Niri/WirePlumber toggling) are tracked in the project's GitHub repo rather than pasted here: [GPU-Passthrough](https://github.com/Mr-Tinkerer/Project-Dump/tree/main/Daily%20Driver%20Laptop/GPU%20Passthrough).

---

Next: [[Pt 7, Automating-with-libvirt-hooks|Pt 7, Automating with libvirt hooks]].
