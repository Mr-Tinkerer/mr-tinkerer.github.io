---
title: OpenMediaVault SSH Hardening
description: Locking down SSH access to the NAS with key-only auth and a non-default port.
tags:
  - Homelab
  - OpenMediaVault
  - Networking
---

## Overview

This page documents SSH access hardening for OpenMediaVault after initial setup, using key-based authentication and a non-standard port.

## Implementation

All configuration is applied through OMV's web interface at `Services → SSH` rather than editing `sshd_config` directly.

### Security Measures

- **SSH Key Authentication** — A dedicated SSH key is issued for logging into the NAS
- **Password Authentication Disabled** — Password login over SSH is disabled, requiring key-based access only
- **Non-Default Port** — The SSH port is changed from the default `22` to `4598`
