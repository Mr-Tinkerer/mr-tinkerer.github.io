---
title: NAS Backup Glossary
description: Terms used across the NAS-Backup bucket (consuming NAS storage from the daily driver).
tags:
  - Glossary
---
# Glossary

## AutoFS
A Linux automounter daemon that mounts network/removable filesystems on demand (the first time something actually accesses the mount point) and unmounts them again after a period of inactivity, rather than keeping them mounted permanently at boot. I use it for the NAS share specifically so the laptop isn't trying to reach the NAS over the network at every boot/login regardless of whether I'm actually on the home network — see [[Automated-document-backup]] for the map files and the one gotcha I hit with it. See [Gentoo Wiki: AutoFS](https://wiki.gentoo.org/wiki/AutoFS).

## NFS
Network File System — a protocol that lets a remote directory tree on another machine (here, the home NAS) be mounted and used as if it were a local folder, with normal file operations working transparently over the network. It's the protocol behind the `/mnt/OMV` share this laptop mounts, and is what [[Nas-backup-glossary#AutoFS|AutoFS]] is actually mounting on demand. See [Wikipedia: Network File System](https://en.wikipedia.org/wiki/Network_File_System).

## rsync
A file-copying/synchronization tool that compares source and destination and only transfers data that actually changed, rather than re-copying everything on every run. That makes it cheap to run on a schedule, which is why it's used here to mirror `~/Documents` to the NAS every few hours instead of a full copy each time. See [rsync.samba.org](https://rsync.samba.org/).

## systemd timer
A systemd unit type (`.timer`) that triggers a paired `.service` unit on a schedule (e.g. hourly), used here instead of cron. See [Arch Wiki: systemd/Timers](https://wiki.archlinux.org/title/Systemd/Timers).
