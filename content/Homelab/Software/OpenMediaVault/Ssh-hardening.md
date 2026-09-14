---
title: SSH hardening
description: Locking down SSH access to the NAS with key-only auth and a non-default port.
tags:
  - Homelab
  - OpenMediaVault
  - Networking
---
I hardened SSH access to the NAS after initial setup:

- I issued a dedicated SSH key for logging into the NAS.
- I disabled password login over SSH, requiring the key for access.
- I changed the SSH port from the default `22` to `4598`.

I applied all of these through OMV's `Services → SSH` settings page rather than editing `sshd_config` directly.
