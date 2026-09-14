---
title: Fixing SPICE Auto-Resize Resolution Mismatches on Wayland
description: Why a SPICE-viewed VM window rendered at double the expected resolution under Niri, and how I forced 1:1 scaling.
tags:
  - Virtual_Machines
  - SPICE
  - Wayland
  - Troubleshooting
---
# Fixing SPICE Auto-Resize Resolution Mismatches on Wayland

When I ran this VM through `virt-manager`/`virt-viewer` (SPICE display, not [[Windows-gaming-vm-glossary#Looking Glass|Looking Glass]]) inside [[Gigabyte-gaming-a16-ga6h-glossary#Niri|Niri]] with fractional display scaling enabled (`1.25`), the guest auto-resized to a resolution much larger than my host window. A window sized to 1221×912 forced the Windows 10 guest into a 2442×1684 canvas.

This only applies to SPICE-based viewing of the VM — it's a separate display path from Looking Glass (Looking Glass reads the passthrough GPU's frames directly via IVSHMEM, see [[GPU-Passthrough/Pt 5, Setting-up-looking-glass]]), so this fix is only relevant while I'm using `virt-viewer`/`virt-manager`'s built-in SPICE console instead of, or before switching to, Looking Glass.

## What was happening

1. **Fractional scaling limits:** GTK3 (which powers standard SPICE clients) doesn't natively support true fractional surface scaling (like `1.25x`) under native Wayland — it only understands integer scaling factors (`1x`, `2x`, `3x`).
2. **The 2x rounding behavior:** when Niri reports a display scale of `1.25`, GTK's native Wayland backend rounds up to the nearest integer — `2.0`.
3. **The canvas multiplier:** the SPICE client reads this rounded `2.0x` scaling factor, multiplies my logical window dimensions (`1221 × 2 = 2442`), and instructs the SPICE agent inside the guest to render at that inflated resolution.

Because the guest was told to render twice as many physical pixels as my layout actually expected, text appeared tiny, blurry, or misaligned with the window bounds.

## The fix

I force the SPICE client to bypass native Wayland scaling and fall back to standard pixel mapping via XWayland, while explicitly clamping the scale multiplier to `1`:

```bash
GDK_BACKEND=x11 GDK_SCALE=1 GDK_DPI_SCALE=1 virt-viewer -c qemu:///system your-vm-name
```

To make this apply every time I launch a VM from the `virt-manager` GUI instead of the terminal, I copied its desktop entry into my user directory so I could edit it without touching the system-wide file:

```bash
cp /usr/share/applications/virt-manager.desktop ~/.local/share/applications/
```

Then I edited the `Exec=` line in `~/.local/share/applications/virt-manager.desktop`:

```ini
Exec=env GDK_BACKEND=x11 GDK_SCALE=1 GDK_DPI_SCALE=1 virt-manager
```

**What each flag does:**
- `GDK_BACKEND=x11` — tells the GTK app to talk to XWayland rather than Niri's native Wayland protocols. XWayland feeds unscaled, exact pixel dimensions down to the app.
- `GDK_SCALE=1` / `GDK_DPI_SCALE=1` — strictly forbids GTK from automatically multiplying its bounding boxes by 2, keeping a true 1:1 ratio between my host layout and the guest's viewport.
