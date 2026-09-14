---
title: Fixing Dark Theme Inconsistencies on Niri
description: Why apps launched from a launcher can render with the wrong theme under Niri, and how I fixed it permanently via environment.d.
tags:
  - Niri
  - Theming
  - Troubleshooting
---
# Fixing Dark Theme Inconsistencies on Niri

> [!summary] TL;DR
> Apps I launch from my **app launcher** don't see the same environment variables as apps I launch from a **terminal**. This caused some apps to randomly render with the wrong (light) theme, black text, or broken icons — even though my config says "dark mode" everywhere. The permanent fix is a file at `~/.config/environment.d/qt-platform.conf`, followed by a **full reboot** (not just logout/login).

---

## Why this happens on Niri

Desktop environments like KDE Plasma or GNOME have a background service that applies theme settings to every app automatically. [[Gigabyte-gaming-a16-ga6h-glossary#Niri|Niri]] doesn't — it's minimal by design, so theming is left up to me.

This causes a specific, sneaky problem: the same app can look different depending on how I launch it. There are three different "launch paths" on my system, and each one can have a *different* set of environment variables:

| Launch method | Where it gets its environment from |
|---|---|
| Typing a command in a terminal | My shell config (`.bashrc`/`.zshrc`) |
| Niri's own shortcuts / `spawn-at-startup` | Niri's own `environment {}` block in its config |
| Clicking an icon in an app launcher | The systemd user session (a background service that manages my login session) |

If a setting like `QT_QPA_PLATFORMTHEME` (which tells Qt apps like Dolphin how to theme themselves) is only set in my terminal or in Niri's config, launcher-opened apps never see it — they fall back to some default, which is often wrong.

> [!info] Why the launcher path is different
> Clicking an icon doesn't ask Niri to run the program directly — it goes through systemd/D-Bus activation, a separate background system that starts apps on my behalf. That system only reads environment variables from one specific place: the [[Gigabyte-gaming-a16-ga6h-glossary#systemd user session|systemd user session]]. Niri's config and my shell's `.bashrc` don't write to that place at all.

---

## Symptom 1: Thunar icons were black/invisible

**Cause:** the icon theme was set to `breeze` (a light-mode icon set) instead of `breeze-dark`, via [[Gigabyte-gaming-a16-ga6h-glossary#gsettings|gsettings]] — a separate settings database that GTK apps (like Thunar) read from, independent of any theme files.

**Fix:**
```bash
gsettings set org.gnome.desktop.interface icon-theme 'breeze-dark'
gsettings set org.gnome.desktop.interface gtk-theme 'Breeze'
```

> [!tip]
> If a theme-switcher tool (like a GTK/Qt theme picker in my desktop shell) is installed, it can silently change these `gsettings` values without me realizing it. If icons or GTK apps suddenly look wrong after "just messing around" in a settings app, I check `gsettings` first.

---

## Symptom 2: Dolphin's text was black on a dark background

This one took longer to track down because it had two separate causes stacked on top of each other.

### Cause A — Dolphin's own color scheme setting can silently reset

This is a known, currently-unresolved bug in Dolphin (KDE's file manager) when it's run outside a full KDE Plasma session. Sometimes Dolphin forgets its saved `ColorScheme=BreezeDark` setting and falls back to a default light-ish scheme.

**Workaround** — I force it every time Dolphin launches, by adding this to my shell config:
```bash
dolphin() {
    kwriteconfig6 --file dolphinrc --group UiSettings --key ColorScheme BreezeDark
    command dolphin "$@"
}
```

### Cause B — the real fix: `QT_QPA_PLATFORMTHEME` wasn't reaching every launch path

Dolphin is a [[Gigabyte-gaming-a16-ga6h-glossary#Qt|Qt]] app, and Qt decides how to theme an app based on the `QT_QPA_PLATFORMTHEME` environment variable. I found:

- From a terminal, this variable was set to [[Gigabyte-gaming-a16-ga6h-glossary#qt6ct|qt6ct]] → text rendered correctly.
- From the app launcher, Qt reported it was actually seeing `gtk3` instead → this plugin mis-renders text color for KDE apps like Dolphin, causing the black-text bug (and was also why the music player Strawberry randomly flipped to light mode).

**The real fix — I set the variable at the systemd level, so every launch path sees it:**

```bash
mkdir -p ~/.config/environment.d
cat > ~/.config/environment.d/qt-platform.conf << 'EOF'
QT_QPA_PLATFORMTHEME=qt6ct
QT_QPA_PLATFORMTHEME_QT6=qt6ct
EOF
```

Then I **fully rebooted** the computer (not just logged out/in — a stale systemd session can survive a normal logout and won't pick up the new file).

**Verifying it worked:**
```bash
systemctl --user show-environment | grep -i QT_QPA
```
I expect to see both variables listed. If they show up here, every app — terminal, launcher, or WM-spawned — now uses the correct theme engine.

> [!warning] Niri's own `environment {}` block is NOT enough
> Setting `QT_QPA_PLATFORMTHEME` inside Niri's config only affects apps Niri directly starts — not apps opened via my launcher/D-Bus. I need [[Gigabyte-gaming-a16-ga6h-glossary#environment.d|environment.d]] on its own, since it covers everything.

---

## Symptom 3: Dolphin's "Open With" dialog showed no applications at all

When trying to open a file with a specific program, Dolphin's picker came up completely empty — not even common apps like a text editor or image viewer showed up.

**Cause:** Dolphin builds its "Open With" list using KDE's own app index (`kbuildsycoca6`), which needs to know which menu definition file to read. On a real Plasma session, Plasma automatically sets an environment variable called [[Gigabyte-gaming-a16-ga6h-glossary#XDG_MENU_PREFIX|XDG_MENU_PREFIX]] at login so the indexer knows where to look. Outside Plasma (on Niri), that variable is never set — so the indexer falls back to looking for a generic file called `applications.menu`, but on CachyOS that file is actually named `plasma-applications.menu`. Since the generic name doesn't exist, the lookup silently fails and the list comes back empty.

**Fix — same pattern as before: I set it via `environment.d` so every launch path sees it:**

```bash
mkdir -p ~/.config/environment.d
cat > ~/.config/environment.d/kde-menu.conf << 'EOF'
XDG_MENU_PREFIX=plasma-
EOF
```

I rebuild KDE's app cache so it picks up the change:
```bash
kbuildsycoca6 --noincremental
```

Then I **reboot fully** — logging out isn't enough, since a stale systemd session can keep ignoring the new `environment.d` file (same as the Qt platform theme issue above).

**Verify after reboot:**
```bash
systemctl --user show-environment | grep XDG_MENU_PREFIX
```

> [!tip] Testing without rebooting first
> I can sanity-check the fix immediately in my current terminal, before committing to a reboot, by setting the variable temporarily:
> ```bash
> export XDG_MENU_PREFIX=plasma-
> kbuildsycoca6 --noincremental
> ```
> If `kbuildsycoca6` runs without a "not found" error and Dolphin's "Open With" list populates in that same terminal session, the fix is confirmed — the reboot is just what's needed to make it permanent and available from every launch path (launcher included).

> [!info] Confirming the actual menu file name
> The `plasma-` prefix is specific to how CachyOS names its menu file. I confirm it with:
> ```bash
> ls /etc/xdg/menus/
> ```
> and use whatever prefix actually matches the filename I see there.

> [!warning] Moving this home partition to a different distro
> `~/.config/environment.d/kde-menu.conf` lives on my home partition, so it'll follow me to a new distro automatically — but the `plasma-` value is specific to how Arch/CachyOS names its menu file. Other distros may use a different name (or no prefix at all). If I ever attach this home partition elsewhere, I'll check `ls /etc/xdg/menus/` again on the new system and update the value if it doesn't match — otherwise the "Open With" dialog can break again there instead.
>
> The KDE app cache itself (`ksycoca`) doesn't need manual rebuilding after a distro switch — KDE detects the mismatch (different installed apps/Frameworks version) and rebuilds it automatically the first time a KDE app runs.

> [!note] Do I need to re-run `kbuildsycoca6` after every install/remove?
> No. Installing/removing a package normally updates its `.desktop` file automatically, and KDE's cache refreshes itself on the next launch. The manual `kbuildsycoca6 --noincremental` was only needed here as a one-time fix for the missing `XDG_MENU_PREFIX` — once that's set persistently, future installs/removals should show up or disappear from "Open With" without any manual steps. If a newly installed app ever doesn't appear, running `kbuildsycoca6 --noincremental` by hand is a safe fallback, just not something I should need routinely.

---

## Quick troubleshooting checklist (for next time something looks "randomly" wrong)

- Does the problem happen with **one launch method but not another** (terminal vs. launcher vs. desktop icon)? → environment variable propagation issue. Check `environment.d`.
- Is it a **GTK app** (Thunar, Nautilus, etc.) with wrong icons/colors? → check `gsettings get org.gnome.desktop.interface icon-theme` / `gtk-theme`.
- Is it a **Qt/KDE app** (Dolphin, Okular, Strawberry, etc.) with wrong colors/text? → check `echo $QT_QPA_PLATFORMTHEME` in a terminal, then compare against `systemctl --user show-environment | grep QT_QPA`. If they differ, that's the bug.
- Is Dolphin's **"Open With" dialog empty**? → check `XDG_MENU_PREFIX` is set (Symptom 3 above).
- After fixing an `environment.d` file, **reboot fully** — don't assume logout/login is enough.
