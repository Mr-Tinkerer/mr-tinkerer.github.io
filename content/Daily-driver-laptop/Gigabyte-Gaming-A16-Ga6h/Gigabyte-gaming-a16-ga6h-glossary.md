---
title: Gigabyte Gaming A16 Ga6h Glossary
description: Terms used across the Gigabyte Gaming A16 Ga6h hardware bucket.
tags:
  - Glossary
---
# Glossary

## udev
A Linux kernel subsystem/daemon that manages device nodes and reacts to hardware events (plugging/unplugging devices, state changes) via rules files. See the [Arch Wiki entry on udev](https://wiki.archlinux.org/title/Udev).

## udev rule
A line in a `.rules` file under `/etc/udev/rules.d/` that matches a device event (by subsystem, environment variable, etc.) and runs an action, such as executing a script. Rules are how this vault's [[Battery-management|battery-management automation]] gets triggered — the rule watches for the AC adapter's `power_supply` device to change `POWER_SUPPLY_ONLINE` state and fires a script in response, with no polling loop needed.

## ACPI power_supply
The Linux kernel class (`power_supply`) exposing battery and AC adapter state (charge, online status, capacity) to userspace via sysfs, sourced from ACPI. See the [kernel power supply class documentation](https://www.kernel.org/doc/html/latest/power/power_supply_class.html).

## power-profiles-daemon
A system daemon that exposes a small set of power profiles (`performance`, `balanced`, `power-saver`) and lets other tools switch between them via `powerprofilesctl`, without each tool having to know how to tweak CPU governors/turbo/thermal limits itself. My [[Battery-management|battery-management script]] calls `powerprofilesctl set <profile>` directly as one of the three things it flips when the AC adapter's state changes. See the [power-profiles-daemon project](https://gitlab.freedesktop.org/hadess/power-profiles-daemon).

## DMS (Dank Material Shell)
The desktop shell I use on this laptop (running on top of [[Gigabyte-gaming-a16-ga6h-glossary#Niri|Niri]]), which also exposes its own CLI (`dms`) for controlling monitor brightness and output/display profiles — functionality a bare window manager like Niri doesn't provide on its own. My [[Battery-management|battery-management script]] shells out to `dms brightness set` and `dms ipc outputs setProfile` to adjust these automatically on plug/unplug. See [danklinux.com](https://danklinux.com/).

## Niri
A scrollable-tiling Wayland compositor/window manager — this laptop's window manager, replacing a traditional desktop environment like GNOME or KDE Plasma. Being minimal by design means it doesn't apply theming or manage per-app environment variables the way a full DE would, which is the root cause behind [[Niri-dark-theme-fix|the dark-theme inconsistencies]] I had to fix manually, and it's also why the [[../Windows-Gaming-VM/GPU-Passthrough/Pt 6, Dynamic-driver-switching|GPU passthrough driver-switching script]] has to explicitly tell Niri to stop rendering through the dedicated GPU rather than that happening automatically. See the [Niri GitHub repository](https://github.com/niri-wm/niri).

## GTK
The toolkit used by apps like Thunar, Nautilus, and GIMP to draw their UI. GTK apps read theme/icon settings from [[Gigabyte-gaming-a16-ga6h-glossary#gsettings|gsettings]] rather than from a Qt-style config file, which is why they can go out of sync with the rest of a Qt-themed desktop — see [[Niri-dark-theme-fix|the dark theme fix]] for a case where this happened. See [gtk.org](https://www.gtk.org/).

## Qt
The toolkit used by apps like Dolphin, Okular, Strawberry, and VLC. Qt apps theme themselves based on the `QT_QPA_PLATFORMTHEME` environment variable rather than gsettings, which matters on a minimal WM like Niri where nothing sets that variable automatically for every launch path. See [qt.io](https://www.qt.io/).

## gsettings
A small settings database GTK apps check for things like icon/theme names (`gsettings get org.gnome.desktop.interface icon-theme`). On Niri there's no background service keeping this in sync with a Qt-side theme choice, so it's a common source of "GTK apps look wrong" symptoms — see [[Niri-dark-theme-fix]]. See [GNOME: dconf/gsettings](https://docs.gtk.org/gio/class.Settings.html).

## qt6ct
A settings app that lets me manually configure themes for Qt apps outside of a full KDE Plasma session — necessary on Niri since nothing else applies Qt theming automatically. See the [qt6ct AUR/project page](https://github.com/trialuser02/qt6ct).

## XDG_MENU_PREFIX
Tells KDE's app-indexing tool (`kbuildsycoca6`) which menu definition file to read when building lists like Dolphin's "Open With" dialog. A full Plasma session sets this automatically at login; Niri doesn't, which silently breaks that dialog until it's set manually via `environment.d` — see [[Niri-dark-theme-fix]]. See the [freedesktop.org menu spec](https://specifications.freedesktop.org/menu-spec/menu-spec-latest.html).

## environment.d
A folder (`~/.config/environment.d/`) where I can place `.conf` files that get loaded into my [[Gigabyte-gaming-a16-ga6h-glossary#systemd user session|systemd user session]] at login — the one environment store shared by every app on Niri, no matter how it's launched (terminal, launcher, or WM-spawned). See [systemd.environment-generator man page](https://www.freedesktop.org/software/systemd/man/latest/environment.d.html).

## systemd user session
A background service that manages my logged-in session and starts apps on my behalf when I click things in a launcher, via systemd/D-Bus activation. On Niri (unlike a full DE) this is the only launch path that reliably shares environment variables across every app, which is why `environment.d` — not Niri's own config — is the right place to fix cross-app environment issues. See [systemd: user session](https://wiki.archlinux.org/title/Systemd/User).
