---
title: Fixing Dark Theme Inconsistencies on Niri
description: Why apps launched from a launcher can render with the wrong theme under Niri, and how it was fixed permanently via environment.d.
tags:
  - Niri
  - Theming
  - Troubleshooting
---

# Fixing Dark Theme Inconsistencies on Niri

> [!summary] TL;DR
> Apps launched from the **app launcher** don't see the same environment variables as apps launched from a **terminal**. This caused some apps to render with the wrong (light) theme, black text, or broken icons — even though the config says "dark mode" everywhere. The permanent fix is a file at `~/.config/environment.d/qt-platform.conf`, followed by a **full reboot** (not just logout/login).

---

## Why this happens on Niri

Desktop environments like KDE Plasma or GNOME have a background service that applies theme settings to every app automatically. [[Gigabyte-gaming-a16-ga6h-glossary#Niri|Niri]] doesn't — it's minimal by design, so theming is left to the user.

This causes a specific, sneaky problem: the same app can look different depending on how it's launched. There are three different "launch paths" on the system, and each one can have a *different* set of environment variables:

| Launch method | Where it gets its environment from |
|---------------|-----------------------------------|
| Typing a command in a terminal | Shell config (`.bashrc`/`.zshrc`) |
| Niri's own shortcuts / `spawn-at-startup` | Niri's own `environment {}` block in its config |
| Clicking an icon in an app launcher | The systemd user session (a background service that manages the login session) |

If a setting like `QT_QPA_PLATFORMTHEME` (which tells Qt apps like Dolphin how to theme themselves) is only set in the terminal or in Niri's config, launcher-opened apps never see it — they fall back to some default, which is often wrong.

> [!info] Why the launcher path is different
> Clicking an icon doesn't ask Niri to run the program directly — it goes through systemd/D-Bus activation, a separate background system that starts apps on behalf of the user. That system only reads environment variables from one specific place: the [[Gigabyte-gaming-a16-ga6h-glossary#systemd user session|systemd user session]]. Niri's config and the shell's `.bashrc` don't write to that place at all.

---

## Symptom 1: Thunar icons were black/invisible

**Cause:** The icon theme was set to `breeze` (a light-mode icon set) instead of `breeze-dark`, via [[Gigabyte-gaming-a16-ga6h-glossary#gsettings|gsettings]] — a separate settings database that GTK apps (like Thunar) read from, independent of any theme files.

**Fix:**
```bash
gsettings set org.gnome.desktop.interface icon-theme 'breeze-dark'
gsettings set org.gnome.desktop.interface gtk-theme 'Breeze'
```

> [!tip]
> If a theme-switcher tool (like a GTK/Qt theme picker in the desktop shell) is installed, it can silently change these `gsettings` values without realizing it. If icons or GTK apps suddenly look wrong after "just messing around" in a settings app, check `gsettings` first.

---

## Symptom 2: Dolphin's text was black on a dark background

This one took longer to track down because it had two separate causes stacked on top of each other.

### Cause A — Dolphin's own color scheme setting can silently reset

This is a known, currently-unresolved bug in Dolphin (KDE's file manager) when it's run outside a full KDE Plasma session. Sometimes Dolphin forgets its saved `ColorScheme=BreezeDark` setting and falls back to a default light-ish scheme.

**Workaround** — Force it every time Dolphin launches, by adding this to the shell config:
```bash
dolphin() {
    kwriteconfig6 --file dolphinrc --group UiSettings --key ColorScheme BreezeDark
    command dolphin "$@"
}
```

### Cause B — the real fix: `QT_QPA_PLATFORMTHEME` wasn't reaching every launch path

Dolphin is a [[Gigabyte-gaming-a16-ga6h-glossary#Qt|Qt]] app, and Qt decides how to theme an app based on the `QT_QPA_PLATFORMTHEME` environment variable. It was found:

- From a terminal, this variable was set to [[Gigabyte-gaming-a16-ga6h-glossary#qt6ct|qt6ct]] → text rendered correctly
- From the app launcher, Qt reported it was actually seeing `gtk3` instead → this plugin mis-renders text color for KDE apps like Dolphin, causing the black-text bug (and was also why the music player Strawberry randomly flipped to light mode)

**The real fix — set the variable at the systemd level, so every launch path sees it:**

```bash
mkdir -p ~/.config/environment.d
cat > ~/.config/environment.d/qt-platform.conf << 'EOF'
QT_QPA_PLATFORMTHEME=qt6ct
QT_QPA_PLATFORMTHEME_QT6=qt6ct
EOF
```

Then **fully reboot** the computer (not just log out/in — a stale systemd session can survive a normal logout and won't pick up the new file).

**Verifying it worked:**
```bash
systemctl --user show-environment | grep -i QT_QPA
```

Expect to see both variables listed. If they show up here, every app — terminal, launcher, or WM-spawned — now uses the correct theme engine.

> [!warning] Niri's own `environment {}` block is NOT enough
> Setting `QT_QPA_PLATFORMTHEME` inside Niri's config only affects apps Niri directly starts — not apps opened via the launcher/D-Bus. It needs [[Gigabyte-gaming-a16-ga6h-glossary#environment.d|environment.d]] on its own, since it covers everything.

---

## Quick troubleshooting checklist (for next time something looks "randomly" wrong)

- Does the problem happen with **one launch method but not another** (terminal vs. launcher vs. desktop icon)? → environment variable propagation issue. Check `environment.d`.
- Is it a **GTK app** (Thunar, Nautilus, etc.) with wrong icons/colors? → check `gsettings get org.gnome.desktop.interface icon-theme` / `gtk-theme`.
- Is it a **Qt/KDE app** (Dolphin, Okular, Strawberry, etc.) with wrong colors/text? → check `echo $QT_QPA_PLATFORMTHEME` in a terminal, then compare against `systemctl --user show-environment | grep QT_QPA`. If they differ, that's the bug.
- After fixing an `environment.d` file, **reboot fully** — don't assume logout/login is enough.
