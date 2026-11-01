---
title: Automatic Battery Management
description: A udev-triggered script that switches power profile, brightness, and display profile when the laptop's AC adapter is plugged or unplugged.
tags:
  - Hardware
  - Automation
  - Bash_Script
---

## Overview

Manually toggling the power profile, monitor brightness, and display profile every time the laptop is plugged/unplugged is easy to forget. This automates it with a udev rule that fires a script whenever the AC adapter's online state changes.

This is documented as hardware content (not software) because its entire purpose is the physical laptop's power/battery behavior — the script only exists to serve the battery, per the domain's hardware-vs-software decision rule.

## Detecting Plug/Unplug State

Watching `sudo udevadm monitor` while plugging/unplugging the charger shows kernel/udev events for two power_supply devices: `ACAD` (the AC adapter) and `BAT1` (the battery).

Running `sudo udevadm info --path=/sys/class/power_supply/ACAD` shows the relevant environment variable: `POWER_SUPPLY_ONLINE`, which is `1` when plugged in and `0` when unplugged.

## The Udev Rule

The rule lives in `/etc/udev/rules.d/80.power.rules`:

```
SUBSYSTEM=="power_supply", ENV{POWER_SUPPLY_ONLINE}=="1", RUN+="/opt/scripts/auto_plugged.sh"
SUBSYSTEM=="power_supply", ENV{POWER_SUPPLY_ONLINE}=="0", RUN+="/opt/scripts/auto_unplugged.sh"
```

Two gotchas when writing this rule:
- `RUN+=` executes a file directly, not a shell command line — wrap shell one-liners in `/bin/sh -c '...'` if testing inline
- The value being compared (`"1"` / `"0"`) must be quoted as a string, or the match silently fails

After editing rules, reload them with `sudo udevadm control --reload-rules && sudo udevadm trigger`.

## What the Script Does

The script run on each transition sets three things via power-profiles-daemon and DMS:
- **Power profile** — `performance` when plugged, `power-saver` when unplugged, via `powerprofilesctl set <profile>`
- **Backlight brightness** — Via `dms brightness set backlight:intel_backlight <percent>`
- **Display output profile** — e.g., "Dual Monitors" vs "Single Laptop Mode", via `dms ipc outputs setProfile "<profile>"`

The commands can't run as root (which is how udev invokes the script), so the script drops to the logged-in user. The same `run_as_user` helper documented in Pt 6 of the Gaming VM's GPU-passthrough scripts is reused, rather than duplicating it.

### Script Maintenance

The script couldn't be made to run reliably entirely at the user level, so Google Gemini was asked to write it instead. The maintained version of the script (rewritten to run continuously via `upower --monitor` rather than purely from the udev one-shot rule, since some commands didn't reliably trigger from udev's context) is autolaunched by Niri on login: [charger_handler.sh](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Daily%20Driver%20Laptop/charger_handler.sh).

## Gotchas

- **User session requirements** — Commands invoked from a udev `RUN+=` rule run as root with no user session — anything that needs `DBUS_SESSION_BUS_ADDRESS`/`XDG_RUNTIME_DIR` (like `dms` or `notify-send`) needs the `run_as_user`-style wrapper
- **Reliable re-triggering** — Not every action reliably re-triggers when driven purely from the one-shot udev `RUN+=` handler; the final script instead runs as a long-lived user-level process that watches `upower --monitor` for state changes
