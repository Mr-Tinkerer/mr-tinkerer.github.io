---
title: Proxmox Glossary
description: Terms used across the Proxmox bucket.
tags:
  - glossary
---

## Overview

This page documents terms used across the Proxmox bucket.

## Proxmox VE

A Debian-based type-1 hypervisor with a web management UI. It runs all of the home lab's VMs.

See [Proxmox VE (official site)](https://www.proxmox.com/en/proxmox-virtual-environment/overview).

## Enterprise Repository

Proxmox's default-enabled package repository, intended for customers with a paid support subscription. Without a subscription key, leaving it enabled just produces update warnings every time updates are checked — it's disabled in favor of the `No-Subscription` repository instead.

See [Proxmox package repositories (official docs)](https://pve.proxmox.com/wiki/Package_Repositories).

## LXC

Linux Containers — an OS-level virtualization method where containers share the host's kernel instead of each running their own, unlike a full VM. Proxmox supports LXC containers alongside full VMs; they're lighter-weight and start faster, at the cost of being restricted to the same kernel as the host. This makes GPU passthrough comparatively simple — no IOMMU/PCI passthrough is needed, just bind-mounting the right device nodes.

See [Linux Containers (official site)](https://linuxcontainers.org/).
