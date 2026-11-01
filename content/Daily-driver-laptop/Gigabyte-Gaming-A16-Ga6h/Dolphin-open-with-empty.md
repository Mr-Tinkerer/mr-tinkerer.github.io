---
title: Dolphin "Open With" Dialog Empty on Niri
description: Dolphin's "Open With" picker shows no applications on Niri, and how to fix it via XDG_MENU_PREFIX.
tags:
  - Niri
  - Dolphin
  - Troubleshooting
---

# Dolphin "Open With" Dialog Empty on Niri

When trying to open a file with a specific program, Dolphin's "Open With" picker came up completely empty — not even common apps like a text editor or image viewer showed up.

---

## Cause

Dolphin builds its "Open With" list using KDE's own app index (`kbuildsycoca6`), which needs to know which menu definition file to read.

On a real Plasma session, Plasma automatically sets an environment variable called [[Gigabyte-gaming-a16-ga6h-glossary#XDG_MENU_PREFIX|XDG_MENU_PREFIX]] at login so the indexer knows where to look. Outside Plasma (on Niri), that variable is never set — so the indexer falls back to looking for a generic file called `applications.menu`, but on CachyOS that file is actually named `plasma-applications.menu`.

Since the generic name doesn't exist, the lookup silently fails and the list comes back empty.

---

## Fix

Set the variable via `environment.d` so every launch path sees it:

```bash
mkdir -p ~/.config/environment.d
cat > ~/.config/environment.d/kde-menu.conf << 'EOF'
XDG_MENU_PREFIX=plasma-
EOF
```

Rebuild KDE's app cache so it picks up the change:

```bash
kbuildsycoca6 --noincremental
```

Then **reboot fully** — logging out isn't enough, since a stale systemd session can keep ignoring the new `environment.d` file (same as the Qt platform theme issue).

**Verify after reboot:**

```bash
systemctl --user show-environment | grep XDG_MENU_PREFIX
```

---

## Testing without rebooting first

Sanity-check the fix immediately in the current terminal, before committing to a reboot, by setting the variable temporarily:

```bash
export XDG_MENU_PREFIX=plasma-
kbuildsycoca6 --noincremental
```

If `kbuildsycoca6` runs without a "not found" error and Dolphin's "Open With" list populates in that same terminal session, the fix is confirmed — the reboot is just what's needed to make it permanent and available from every launch path (launcher included).

---

## Confirming the actual menu file name

The `plasma-` prefix is specific to how CachyOS names its menu file. It confirms it with:

```bash
ls /etc/xdg/menus/
```

and uses whatever prefix actually matches the filename it sees there.

---

## Moving this home partition to a different distro

`~/.config/environment.d/kde-menu.conf` lives on the home partition, so it'll follow to a new distro automatically — but the `plasma-` value is specific to how Arch/CachyOS names its menu file. Other distros may use a different name (or no prefix at all).

If this home partition is ever attached elsewhere, check `ls /etc/xdg/menus/` again on the new system and update the value if it doesn't match — otherwise the "Open With" dialog can break again there instead.

The KDE app cache itself (`ksycoca`) doesn't need manual rebuilding after a distro switch — KDE detects the mismatch (different installed apps/Frameworks version) and rebuilds it automatically the first time a KDE app runs.

---

## Do you need to re-run `kbuildsycoca6` after every install/remove?

No. Installing/removing a package normally updates its `.desktop` file automatically, and KDE's cache refreshes itself on the next launch. The manual `kbuildsycoca6 --noincremental` was only needed here as a one-time fix for the missing `XDG_MENU_PREFIX` — once that's set persistently, future installs/removals should show up or disappear from "Open With" without any manual steps.

If a newly installed app ever doesn't appear, running `kbuildsycoca6 --noincremental` by hand is a safe fallback, just not something that should be needed routinely.
