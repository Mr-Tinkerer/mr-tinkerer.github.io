---
title: Sonarr Setup
description: TV show management and automated downloading for the ARR stack.
tags:
  - Homelab
  - Arrrrgh-VM
  - Sonarr
---

## Overview

Sonarr manages automated TV show downloading and organization. It monitors RSS feeds, downloads episodes, and organizes them in the media library.

---

## Installation

1. Install [Sonarr](https://sonarr.tv/) .exe
2. Run executable
3. Allow MS Defender exception
4. Select not to start automatically
5. Switch to admin and launch
6. Set authentication to forums
7. Create username and password
8. Set allowed hosts to `10.10.0.4,127.0.0.1,sonarr.uhhhhh`
9. Restart Sonarr

---

## Configuration

| Setting | Value |
|---------|-------|
| Media Management → File Management → Add Root Folder | `Z:\The Seven Seas\media\tv\` |
| Indexers → Show advanced | Checked |
| Indexers → Indexers → Fail Downloads | Executables, Potentially Dangerous |

---

### Auto-Startup

1. Login as admin
2. Open Task Scheduler
3. Create task named `Start Sonarr on boot`
4. Select "Run whether user is logged on or not"
5. Triggers → New → At startup
6. Actions → New → Start a program
7. Program/Script: `C:\ProgramData\Sonarr\bin\Sonarr.Console.exe`
8. Uncheck "Stop this task if it runs longer than"
9. Click OK (requires admin password)
10. Reboot to test

### Firewall Configuration

Create inbound firewall rule for Sonarr WebUI access:

1. Search Windows Defender Firewall with Advanced Security
2. Inbound rules → New Rule...
3. Rule type: Program
4. Program path: `C:\ProgramData\Sonarr\bin\Sonarr.Console.exe`
5. Select Allow the connection
6. Apply to everyone
7. Name rule: `Sonarr WebUI Access`

---

## Notifications

1. Setup Discord notifications
2. Settings → Notifications
3. Click + button
4. Select Discord
5. Go to Discord server channel
6. Right-click → Edit channel → Integrations → Create webhook
7. Put in name of notification user
8. Copy webhook URL
9. Paste into Sonarr webhook URL field
10. Test and save

---

## Connect to Prowlarr's Indexers

1. Settings → General → Security → Copy API key
2. Go to Prowlarr
3. Settings → Apps
4. Click on Sonarr
5. Paste API key
6. Test and save
7. Click on Sync app indexers
