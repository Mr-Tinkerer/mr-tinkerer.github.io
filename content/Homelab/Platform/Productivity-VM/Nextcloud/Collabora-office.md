---
title: Collabora Office
description: Adding in-browser document editing to Nextcloud via a separate Collabora Online container.
tags:
  - Homelab
  - Nextcloud
---

## Overview

This page covers adding in-browser document editing to Nextcloud via a separate Collabora Online container.

## Architecture

Nextcloud Office is powered by a separate Collabora Online server, run as its own container (`collabora`) on the same Podman network as the rest of the Nextcloud stack.

- **Access:** `office.nextcloud.uhhhhh`
- **Container:** `collabora` on the shared Podman network

See [reachable verification screenshot](Images/nextcloud-collabora-server-reachable-verification.png) for Nextcloud confirming Collabora Online is reachable, along with the URLs each side uses to reach the other.

## Verification

Document editing was confirmed end-to-end by opening and editing a `.docx` file directly in the browser.

## Gotchas

See the Gotchas page for the certificate-trust and Nginx-redirect problems this integration hit before it worked.
