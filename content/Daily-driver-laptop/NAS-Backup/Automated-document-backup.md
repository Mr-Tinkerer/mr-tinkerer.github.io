---
title: Automated Document Backup to the Home NAS
description: Mounting a home NAS share and automatically rsync-ing the laptop's Documents folder to it on a schedule.
tags:
  - Backup
  - Automation
---

# Automated Document Backup to the Home NAS

The laptop consumes network storage from the home NAS (a separate physical device outside this domain) over NFS, and keeps a local Documents folder backed up to it automatically.

---

## Mounting the NAS share

The share is mounted on demand at `/mnt/OMV` using [[Nas-backup-glossary#AutoFS|AutoFS]]. One gotcha encountered: AutoFS doesn't show the share's root directory by default, which made `/mnt/OMV` look empty and initially looked like a broken mount — it wasn't. The [Gentoo Wiki's AutoFS useful-options section](https://wiki.gentoo.org/wiki/AutoFS#Useful_options) documents the option that addresses this.

The AutoFS map files used (`auto.master`, and the `omv.nfs` map with per-share NFS entries) are maintained at [Project-Dump: Daily Driver Laptop/autofs](https://github.com/Mr-Tinkerer/Project-Dump/tree/main/Daily%20Driver%20Laptop/autofs).

## Automatic rsync backup

A systemd service + timer pair automatically syncs the local `~/Documents` folder to the mounted NAS share every 3 hours. This runs alongside, not instead of, the [[../../Failed-laptop/Asus-X555q/Borg-backup-automation|Borg-based backup]] from the old laptop-NAS project (both are still active — see [Project-Dump: Auto Backup](https://github.com/Mr-Tinkerer/Project-Dump/tree/main/Daily%20Driver%20Laptop/Auto%20Backup) for both script/service families) — they cover different recovery scenarios. Borg keeps deduplicated, searchable-by-date snapshots of the whole home directory, but decompressing and searching through them for one specific file can take a while; it's the answer to "just ran `rm -rf *` in my home directory." This rsync pair instead keeps a plain, immediately-browsable mirror of just `~/Documents`; it's the answer to "deleted a document a few hours ago that was thought no longer needed, but it was."

`auto_documents_backup.service` and `auto_documents_backup.timer` are authored configuration, so they are referenced rather than pasted in full per the file-reference rule: [auto_documents_backup.service](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Daily%20Driver%20Laptop/Auto%20Backup/Services/auto_documents_backup.service), [auto_documents_backup.timer](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Daily%20Driver%20Laptop/Auto%20Backup/Services/auto_documents_backup.timer). In short: the service runs `rsync -avh --del --progress ~/Documents /mnt/OMV/Documents` as a `oneshot` unit, logging to the journal, and the [[Nas-backup-glossary#systemd timer|timer]] fires it every 3 hours (`OnCalendar=*-*-* 00/3:00:00`).

## Verifying it's working

- Check `systemctl list-timers` for `auto_documents_backup.timer` showing a sane "next" run time.
- Compare file counts/timestamps between `~/Documents` and `/mnt/OMV/Documents` after a run.
