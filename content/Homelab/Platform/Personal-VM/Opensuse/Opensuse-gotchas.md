---
title: openSUSE Gotchas on Personal VM
description: openSUSE-specific defaults that tripped up getting any container reachable and writable on the Personal VM, independent of which service was actually installed.
tags:
  - Homelab
  - Opensuse
---

## Overview

This page documents openSUSE-specific defaults that affected container accessibility and writability on the Personal VM, regardless of the specific service being installed.

## OS-Level Defaults

Two separate openSUSE default-security surprises affected any containerized service on this VM:

### firewalld

openSUSE ships with `firewalld` enabled, so a freshly deployed container's port was unreachable from the LAN until it was disabled (or, more carefully, the specific port/service was opened in its zone config).

### SELinux

openSUSE ships SELinux enabled and enforcing by default:

See [SELinux enforcing screenshot](Images/personal-vm-selinux-enforcing-sestatus.png) for `sestatus` showing SELinux enabled and enforcing on the Personal VM — the actual cause of a mounted config volume being unwritable, not a bug in whatever container was trying to use it.

**Fix:** Add the `:Z` SELinux relabel suffix to the volume mount in the Podman quadlet, which lets the container write to the mounted directory under SELinux's enforcing policy.

## Scope

Both issues are OS-level defaults, not caused by Homepage or Podman. They would affect any container on this VM, which is why they live here rather than in a specific service's bucket.
