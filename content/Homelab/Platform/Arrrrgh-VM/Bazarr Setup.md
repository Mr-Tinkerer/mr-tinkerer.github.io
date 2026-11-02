---
title: Bazarr Setup
description: Subtitle management service for the ARR stack.
tags:
  - Homelab
  - Arrrrgh-VM
  - Bazarr
---

## Overview

Bazarr downloads subtitles for Sonarr and Radarr libraries based on user preferences and language profiles.

---

## Installation

1. Install [Bazarr](https://www.bazarr.media/) .zip
2. Unzip and run executable
3. Wizard auto-starts on boot
4. Access web UI
5. Set address to `10.10.0.4`
6. Set authentication to forums
7. Create username and password
8. Restart Bazarr

---

### Notifications

1. Settings → Notifications
2. Click + button
3. Select Discord
4. Go to Discord server channel
5. Create webhook
6. Copy webhook URL
7. Paste into Bazarr webhook URL
8. Test and save

---

## Connect to Sonarr and Radarr

1. Settings → Library → Sonarr
2. Enable Sonarr
3. Go to Sonarr
4. Settings → General → Security → Copy API key
5. Return to Bazarr and paste API key
6. Test connection
7. Repeat for Radarr

---

## Profiles

1. Settings → Languages → Selection
2. Configure Languages Filters
3. Specify languages to use (e.g., English)
4. Settings → Languages → Profiles
5. Click Add new Profile
6. Name: Main
7. Click Add Language
8. Confirm
9. Scroll to Default Language Profiles For Newly Added Shows
10. Set series and movies to Main
11. Save

---

## Providers

### Subtitles

Settings → Providers → Subtitles

Used providers:
- Addic7ed.com (requires Anti-Captcha)
  1. Go to addic7ed.com
  2. Press F12 for developer console
  3. Go to Storage tab
  4. Go to Cookies and click `www.addic7ed.com`
  5. Look for `PHPSESSID`
  6. Copy value
  7. Paste `PHPSESSID=<value>` into cookies field
- Anime Toshio (requires Anidb metadata)
- Tsukihime (requires Anidb metadata)
- Getsdown (Addic7ed proxy)

### Metadata

Settings → Providers → Metadata

1. Click + button
2. Select Anidb from list
3. Go to [anidb.net](https://anidb.net)
4. Create account
5. Login to Anidb
6. Go to [api clients](https://anidb.net/software/add)
7. Name it bazarr
8. Click Add projects
9. Click Add client
10. Name it bazarrwinserver
11. Set API to `HTTP API`
12. Set version to 9
13. Click Add client
14. Return to Bazarr
15. Put client name in API client field
16. Set version to 9
17. Enable and save