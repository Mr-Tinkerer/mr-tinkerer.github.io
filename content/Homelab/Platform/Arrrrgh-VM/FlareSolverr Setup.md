---
title: FlareSolverr Setup on Arrrrgh Windows Server
description: CAPTCHA bypass and indexer access tool for the ARR stack.
tags:
  - Homelab
  - Arrrrgh-VM
  - FlareSolverr
---

## Overview

FlareSolverr is a browser automation tool that bypasses CAPTCHA and provides access to restricted indexer websites. It proxies requests to indexers that block direct access.

---

## Installation

1. Download Windows version from [FlareSolverr releases](https://github.com/FlareSolverr/FlareSolverr/releases)
2. Unzip the archive
3. Double-click the executable
4. Service runs on port 8191
5. Move the folder to Documents for persistence

---

## Auto-Startup

1. Login as admin
2. Open Task Scheduler
3. Select Create task
4. Name: `Start FlarSolverr on boot`
5. Select "Run whether user is logged on or not"
6. Triggers tab → New → At startup
7. Actions tab → New → Start a program
8. Program/Script: `C:\Users\Arrrrgh\Documents\flaresolverr\flaresolverr.exe`
9. Uncheck "Stop this task if it runs longer than"
10. Click OK (requires admin password)
11. Reboot to test

---

## Prowlarr Integration

1. Go to Prowlarr Settings → Indexers
2. Click + button
3. Select FlareSolverr
4. Type "flareSolverr" in tags
5. Click Test and wait for green checkmark
6. Click Save
7. Add FlareSolverr as tag to indexers requiring it