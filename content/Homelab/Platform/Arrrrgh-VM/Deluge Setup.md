---
title: Deluge Setup on Arrrrgh Windows Server
description: Torrent client for the ARR stack with label-based organization.
tags:
  - Homelab
  - Arrrrgh-VM
  - Deluge
---

## Overview

Deluge is the torrent client for the ARR stack. It handles BitTorrent and Usenet downloads while Sonarr and Radarr manage download organization and metadata.

---

## Installation

1. Download from [Deluge Windows](https://ftp.osuosl.org/pub/deluge/windows/?C=M;O=D)
2. Run wizard installer
3. Navigate to `C:\Program Files\Deluge`
4. Run `deluge-web.exe` for web server

---

## Web Interface

- Access: `localhost:8112`
- Password: Set during configuration

---

## Configuration

| Setting | Value |
|---------|-------|
| Interface → Theme | Access |
| Interface → WebUI Password | Changed Password |
| Downloads → Download to | `Z:\The Seven Seas\torrents` |
| Downloads → Move completed to | `Z:\The Seven Seas\media` |
| Downloads → Pre-allocate disk space | Checked |
| Queue → Active Torrents → Total | -1 |
| Queue → Active Torrents → Downloading | 5 |
| Network → Network Extras → UPnP | Unchecked |
| Network → Network Extras → NAT-PMP | Unchecked |
| Plugins → Label | Checked |

---

## Labels Configuration

1. Click Labels in filters sidebar
2. Right-click All → Add Label
3. Name first label "TV"
4. Right-click TV label → Label Options → Folders
5. Check "Apply folder settings"
6. Check "Move completed to" and set to `Z:\The Seven Seas\torrents\tv`
7. Click OK
8. Repeat for remaining torrent folders

---

### Auto-Startup

1. Login as admin
2. Open Task Scheduler
3. Create task named `Start Deluge on boot`
4. Select "Run whether user is logged on or not"
5. Triggers → New → At startup
6. Actions → New → Start a program
7. Program/Script: `C:\Program Files\Deluge\deluge-web.exe`
8. Uncheck "Stop this task if it runs longer than"
9. Click OK (requires admin password)
10. Reboot to test
11. Repeat for Deluge daemon: `C:\Program Files\Deluge\deluged.exe`

### Firewall Configuration

Create inbound firewall rule for Deluge WebUI access:

1. Search Windows Defender Firewall with Advanced Security
2. Inbound rules → New Rule...
3. Rule type: Program
4. Program path: `C:\Program Files\Deluge\deluge-web.exe`
5. Select Allow the connection
6. Apply to everyone
7. Name rule: `Deluge WebUI Access`

---

## Connect to ARR Stack

1. Go to any ARR stack
2. Settings → Download client
3. Click + button
4. Select Deluge
5. Host: localhost
6. Port: 8112
7. Password: Changed Password
8. Label: ARR stack role from folder structure
9. Test connection and save
10. Repeat for remaining ARR stack services