---
title: SPICE Auto-Resize Fix on Wayland
description: Why a SPICE-viewed VM window rendered at double the expected resolution under Niri, and how to force 1:1 scaling.
tags:
  - Virtual_Machines
  - SPICE
  - Wayland
  - Troubleshooting
---

## Overview

When a VM was run through virt-manager/virt-viewer (SPICE display, not Looking Glass) inside Niri with fractional display scaling enabled (1.25), the guest auto-resized to a resolution much larger than the host window. A window sized to 1221×912 forced the Windows 10 guest into a 2442×1684 canvas.

This only applies to SPICE-based viewing of the VM — it's a separate display path from Looking Glass (Looking Glass reads the passthrough GPU's frames directly via IVSHMEM), so this fix is only relevant while using virt-viewer/virt-manager's built-in SPICE console instead of, or before switching to, Looking Glass.

## Root Cause

### Fractional Scaling Limits

GTK3 (which powers standard SPICE clients) doesn't natively support true fractional surface scaling (like 1.25x) under native Wayland — it only understands integer scaling factors (1x, 2x, 3x).

### 2x Rounding Behavior

When Niri reports a display scale of 1.25, GTK's native Wayland backend rounds up to the nearest integer — 2.0.

### Canvas Multiplier

The SPICE client reads this rounded 2.0x scaling factor, multiplies the logical window dimensions (1221 × 2 = 2442), and instructs the SPICE agent inside the guest to render at that inflated resolution.

Because the guest was told to render twice as many physical pixels as the layout actually expected, text appeared tiny, blurry, or misaligned with the window bounds.

## The Fix

Force the SPICE client to bypass native Wayland scaling and fall back to standard pixel mapping via XWayland, while explicitly clamping the scale multiplier to 1:

```bash
GDK_BACKEND=x11 GDK_SCALE=1 GDK_DPI_SCALE=1 virt-viewer -c qemu:///system your-vm-name
```

### Persistent Configuration

To make this apply every time a VM is launched from the virt-manager GUI instead of the terminal, the desktop entry was copied into the user directory:

```bash
cp /usr/share/applications/virt-manager.desktop ~/.local/share/applications/
```

Then the `Exec=` line in `~/.local/share/applications/virt-manager.desktop` was edited:

```ini
Exec=env GDK_BACKEND=x11 GDK_SCALE=1 GDK_DPI_SCALE=1 virt-manager
```

### Flag Explanations

- **GDK_BACKEND=x11** — Tells the GTK app to talk to XWayland rather than Niri's native Wayland protocols. XWayland feeds unscaled, exact pixel dimensions down to the app
- **GDK_SCALE=1** / **GDK_DPI_SCALE=1** — Strictly forbids GTK from automatically multiplying its bounding boxes by 2, keeping a true 1:1 ratio between the host layout and the guest's viewport
