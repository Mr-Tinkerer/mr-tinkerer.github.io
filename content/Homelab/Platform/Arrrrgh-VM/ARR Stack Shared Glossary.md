---
title: ARR Stack Shared Glossary
description: Common terms and concepts used across ARR stack services.
tags:
  - Homelab
  - Arrrrgh-VM
  - Prowlarr
  - FlareSolverr
  - Deluge
  - Sonarr
  - Radarr
  - Bazarr
  - glossary
---

## Overview

This glossary defines common terms and concepts used across the ARR stack services.

---

## ARR Stack

App Repository & Remote (ARR) stack refers to a collection of media management services that work together: Prowlarr (indexer management), Deluge (torrent client), Sonarr (TV shows), Radarr (movies), and Bazarr (subtitles).

---

## FlareSolverr

A browser automation tool used to bypass CAPTCHA and access restricted indexer websites. FlareSolverr proxies requests to indexers that block direct access.

---

## NFS (Network File System)

A distributed file system protocol that allows a client to mount and access files on a remote server as if they were local. Used for sharing media libraries between ARR stack services.

---

## Task Scheduler

A Windows service that allows automated execution of programs or scripts at specified times or triggers. Used for auto-starting ARR stack services on system boot.

---

## UID/GID Mapping

User ID (UID) and Group ID (GID) mapping in containerized environments. LXC containers shift UIDs/GIDs by a base offset (typically 100000) to map container users to host users.

---

## API Key

A unique token used for authentication between ARR stack services. Services exchange API keys to enable integration and data sharing.

---

## Indexer

A torrent or usenet tracker that provides access to media files. Indexers can be public or private and may have different access requirements (e.g., CAPTCHA, registration).

---

## Label

A categorization system used in Deluge and other services to organize downloads. Labels can have folder associations for automatic file placement.

---

## Profile

A configuration set for specific functionality. In Bazarr, profiles define language preferences and subtitle download behavior.

---

## Provider

A metadata or subtitle source used by services like Sonarr, Radarr, and Bazarr. Providers can require special authentication (e.g., cookies, API keys).
