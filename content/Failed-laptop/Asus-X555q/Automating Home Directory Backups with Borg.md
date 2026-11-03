---
title: Automating Home Directory Backups with Borg
description: Backing up a client's home directory onto the NAS share with BorgBackup, then automating it with a script and systemd timers instead of running it by hand.
tags:
  - Asus-X555q
  - OpenMediaVault
  - Backup
---

## Overview

This page covers backing up a client's home directory onto the NAS share with BorgBackup, then automating it with a script and systemd timers instead of running it by hand.

## Why Borg

With the NAS share mounted (see [[Auto-Mounting the NAS Share with autofs|Auto-Mounting the NAS Share]]), BorgBackup backs up the client's home directory onto it with compression, encryption, and deduplication of repeated data across backups. It's Unix-only — no Windows support — which isn't a limitation here since the client being backed up is Linux.

## Initial Repository and Manual Backup

A Borg repository was created once, at the mount point, with `borg init --encryption=repokey borg`. A backup was taken with `borg create`, excluding caches, game libraries, and the trash directory, combined with `notify-send` so a desktop notification appears on completion of what can be a multi-hour run.

See [initial backup stats screenshot](Images/borg-backup-initial-run-stats.png) for stats from the first backup run: ~543,000 files, 178.87 GB deduplicated down to 110.39 GB, taking just under 3 hours — evidence of the backup actually completing successfully with real numbers.

## Automating the Backup

Running the `borg create` command by hand isn't sustainable. A bash script wraps it, also validating its prerequisites before running: it checks the Borg repo exists, reads the encryption passphrase from a local secret file (restricting that file to `chmod 400` before use), and sends desktop notifications for start, completion (with duration and size), and pruning old backups beyond a configured maximum count.

The script is maintained in the [Project-Dump repo](https://github.com/Mr-Tinkerer/Project-Dump/tree/main/Daily%20Driver%20Laptop/Auto%20Backup) rather than reproduced here. Using it requires setting the repo path, placing the Borg passphrase in the secret file it expects, and adjusting the include/exclude paths for own backup targets.

See [notification screenshot](Images/borg-backup-automation-notify-send-notifications.png) for desktop notifications firing for the start and completion of an automated run — confirming the script's notification path actually works end-to-end, not just the backup itself.

## Scheduling with systemd Timers

The script runs on a schedule via a systemd user service and timer (rather than cron), because systemd timers can catch up on a missed run if the machine was powered off at the scheduled time, while cron simply skips it. The user-level service/timer also lets desktop notifications reach the logged-in session — this requires the timer's service unit to run with the same `DISPLAY` and `DBUS_SESSION_BUS_ADDRESS` environment values as the active graphical session.

### Service and Timer Files

The service and timer unit files are kept in the [Project-Dump repo](https://github.com/Mr-Tinkerer/Project-Dump/tree/main/Daily%20Driver%20Laptop/Auto%20Backup/Services) rather than pasted here, since they're user-maintained config. In summary:

- The service unit runs the backup script once (`Type=oneshot`)
- The timer unit triggers that service on a recurring schedule with `Persistent=true` set so a missed run fires as soon as the machine is next on

After creating or editing either unit, `systemctl --user daemon-reload` picks up the change, and `systemctl --user enable --now auto_backup.timer` enables and starts the timer. `systemctl --user list-timers` shows the next scheduled run.

## Gotchas

- **User service desktop notifications** — If a systemd user service's desktop notifications never appear, check that `DISPLAY` and `DBUS_SESSION_BUS_ADDRESS` are set explicitly in the unit and match the active graphical session's values — a user service does not automatically inherit them
- **Systemd timers vs cron** — Prefer systemd timers with `Persistent=true` over cron for backup jobs on a machine that isn't always powered on, so a missed scheduled run still happens on next boot instead of being silently skipped
