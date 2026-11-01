---
title: Nextcloud Gotchas
description: Certificate trust, security warnings, and Nginx redirect issues while configuring Nextcloud and Collabora with private CA.
tags:
  - Homelab
  - Nextcloud
  - Gotchas
---

## Overview

This page documents issues encountered during Nextcloud and Collabora configuration with private CA certificates, including security warnings, mutual certificate validation failures, and Nginx redirect errors.

## Security and Setup Warnings

### Initial Warnings

Nextcloud's admin UI flagged multiple errors and warnings after initial installation, before HTTPS and reverse-proxy trust were configured.

See [security warnings screenshot](Images/nextcloud-security-setup-warnings-before-fix.png) for the Security & setup warnings panel showing incorrect forwarded-for headers and insecure HTTP access.

### Resolution

- **Trusted Proxies:** Configure Nextcloud to trust the Nginx reverse proxy and its subnet (`trusted_proxies`)
- **Force HTTPS:** Set `overwriteprotocol` to always assume HTTPS

These settings are baked into the custom Nextcloud image to survive container rebuilds.

## Certificate Validation Issues

### Problem Description

Both Nextcloud and Collabora need to reach each other over HTTPS, but neither trusted the other's certificate since both use the home lab's private CA rather than a public certificate authority.

### Resolution

The private CA's root certificate was manually installed inside both the `nextcloud-custom` and `collabora-custom` container images rather than disabling certificate verification.

### Red Herring Investigation

A Collabora container child process spawning failure appeared to indicate a Podman-specific security restriction (rootless/more restrictive defaults compared to Docker). Testing with Docker did not resolve the issue, confirming the cause was unrelated to the container runtime.

## Nginx Redirect Error

### Root Cause

The actual cause of Collabora connection failures was a copy-paste mistake in Nginx's configuration for the Collabora subdomain:

- File: `office.nextcloud.conf` HTTP→HTTPS block
- Error: Redirecting to `http://` instead of `https://`

### Resolution

The redirect was corrected to use `https://` in the Nginx configuration. See the Nginx bucket for the complete reverse-proxy configuration.

## Mobile App Certificate Trust

The Nextcloud mobile app did not work with the private CA out of the box. Root cause was not further investigated. See the SSL certificates page for a related mobile-browser certificate-trust issue (Waterfox), which is a Nginx/CA-level issue rather than Nextcloud-specific.
