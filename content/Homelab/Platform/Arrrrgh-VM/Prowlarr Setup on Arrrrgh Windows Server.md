---
title: Prowlarr Setup on Arrrrgh Windows Server
description: Indexer management and metadata coordination for the ARR stack.
tags:
  - Homelab
  - Arrrrgh-VM
  - Prowlarr
---

## Overview

Prowlarr manages indexer settings and provides API integration for the ARR stack services. It maintains a centralized list of available torrent indexers and coordinates metadata retrieval.

---

## Authentication

1. Visit [Prowlarr website](https://prowlarr.com/)
2. Download the Windows executable
3. Launch the Prowlarr site
4. Set authentication to forums
5. Create a username and password for the account
6. Set allowed hosts to `10.10.0.4,127.0.0.1,prowlarr.uhhhhh`
7. Restart Prowlarr to apply changes

### Auto-Startup

1. Login as admin
2. Open Task Scheduler
3. Select Create task
4. Name: `Start Prowlarr on boot`
5. Select "Run whether user is logged on or not"
6. Triggers → New → At startup
7. Actions → New → Start a program
8. Program/Script: `C:\ProgramData\Prowlarr\bin\Prowlarr.Console.exe`
9. Uncheck "Stop this task if it runs longer than"
10. Click OK (requires admin password)
11. Reboot to test

### Firewall Configuration

Create inbound firewall rule for Prowlarr WebUI access:

1. Search Windows Defender Firewall with Advanced Security
2. Inbound rules → New Rule...
3. Rule type: Program
4. Program path: `C:\ProgramData\Prowlarr\bin\Prowlarr.Console.exe`
5. Select Allow the connection
6. Apply to everyone
7. Name rule: `Prowlarr WebUI Access`

---

## Indexer Configuration

1. Visit [Prowlarr website](https://prowlarr.com/)
2. Download the Windows executable
3. Launch the Prowlarr site
4. Set authentication to forums
5. Create a username and password for the account
6. Set allowed hosts to `10.10.0.4,127.0.0.1`
7. Restart Prowlarr to apply changes

---

## Indexer Configuration

Add the following indexers:

| Indexer | FlareSolverr Required |
|---------|----------------------|
| 1337x | Yes |
| EZTV | Yes |
| Nyaa.si | No |
| LimeTorrents | No |
| The Pirate Bay | No |

Indexers to try in the future:
- EXT.TO
- TheRARBG

### Indexer Priority

Enable advanced mode for indexer priority:

1. Settings → General → Show advanced
2. Configure download priority:
   - 1337x: 25
   - EZTV: 25
   - Nyaa.si: 20
   - LimeTorrents: 30
   - The Pirate Bay: 25

---

## Connect to ARR Stack

1. Go to any ARR stack web interface
2. Settings → General → Security → Copy API key
3. Settings → Apps
4. Click on the ARR stack
5. Paste API key
6. Test and save
7. Click on Sync app indexers
8. Repeat for remaining ARR stack services