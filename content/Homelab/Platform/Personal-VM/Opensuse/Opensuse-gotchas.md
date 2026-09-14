---
title: openSUSE Gotchas
description: openSUSE-specific defaults that tripped up getting any container reachable and writable on the Personal VM, independent of which service I was actually installing.
tags:
  - Homelab
  - Opensuse
---
# openSUSE ships firewalld and SELinux enabled by default

I hit two separate openSUSE default-security surprises while bringing up the Personal VM, both of which would apply to *any* containerized service I put on this VM, not just [[Homelab/Platform/Personal-VM/Homepage/Homepage-setup|Homepage]] (which is what actually surfaced them for me first):

1. **[[Homelab/Platform/Personal-VM/Opensuse/Opensuse-glossary#firewalld|`firewalld`]] blocks ports by default.** openSUSE ships with `firewalld` enabled, so a freshly deployed container's port was unreachable from the LAN until I disabled it (or, more carefully, opened the specific port/service in its zone config).
2. **[[Homelab/Platform/Personal-VM/Opensuse/Opensuse-glossary#SELinux|SELinux]] blocks container writes into mounted volumes.** openSUSE also ships SELinux enabled and enforcing by default:

   ![[personal-vm-selinux-enforcing-sestatus.png]]
   *`sestatus` showing SELinux enabled and enforcing on the Personal VM — the actual cause of a mounted config volume being unwritable, not a bug in whatever container is trying to use it.*

   **Fix:** add the [[Homelab/Platform/Personal-VM/Opensuse/Opensuse-glossary#`:Z` volume relabel suffix|`:Z` SELinux relabel suffix]] to the volume mount in the Podman quadlet, which lets the container write to the mounted directory under SELinux's enforcing policy.

Both are OS-level defaults, not something caused by Homepage or by Podman — I'd hit the same two issues bringing up any container on this VM, which is why they live here rather than in a specific service's bucket.
