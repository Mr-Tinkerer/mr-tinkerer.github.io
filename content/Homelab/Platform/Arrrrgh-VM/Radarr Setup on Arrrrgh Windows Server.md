---
title: Radarr Setup on Arrrrgh Windows Server
description: Movie management and automated downloading for the ARR stack.
tags:
  - Homelab
  - Arrrrgh-VM
  - Radarr
---

## Overview

Radarr manages automated movie downloading and organization. It mirrors Sonarr's functionality but for movies.

---

## Installation

1. Install [Radarr](https://radarr.video/) .exe
2. Run executable
3. Allow MS Defender exception
4. Select not to start automatically
5. Switch to admin and launch
6. Set authentication to forums
7. Create username and password
8. Set allowed hosts to `10.10.0.4,127.0.0.1,radarr.uhhhhh`
9. Restart Radarr

---

## Configuration

| Setting | Value |
|---------|-------|
| Media Management → File Management → Add Root Folder | `Z:\The Seven Seas\media\movies\` |
| Indexers → Show advanced | Checked |
| Indexers → Indexers → Fail Downloads | Executables, Potentially Dangerous |

---

### Auto-Startup

Same procedure as Sonarr, with Program/Script: `C:\ProgramData\Radarr\bin\Radarr.Console.exe`

### Firewall Configuration

Create inbound firewall rule for Radarr WebUI access:

1. Search Windows Defender Firewall with Advanced Security
2. Inbound rules → New Rule...
3. Rule type: Program
4. Program path: `C:\ProgramData\Radarr\bin\Radarr.Console.exe`
5. Select Allow the connection
6. Apply to everyone
7. Name rule: `Radarr WebUI Access`

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
9. Paste into Radarr webhook URL field
10. Test and save

---

## Connect to Prowlarr's Indexers

1. Settings → General → Security → Copy API key
2. Go to Prowlarr
3. Settings → Apps
4. Click on Radarr
5. Paste API key
6. Test and save
7. Click on Sync app indexers