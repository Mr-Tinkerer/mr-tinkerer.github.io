---
title: Personal-VM Overview
description: Overview of the Personal-VM, its role in the home lab, and the services it hosts.
tags:
  - Homelab
  - Platform
  - Personal-VM
---

## Overview

This page documents the Personal-VM, its role in the home lab, and the services it hosts.

## Personal-VM Role and Purpose

The Personal-VM provides a dedicated openSUSE-based platform for personal computing and web browsing. It runs on Proxmox as a LXC container configured as a full system container for desktop application support.

## Guest OS

The VM runs openSUSE as its guest operating system, configured with appropriate resource allocation for desktop workloads. See [[Homelab/Platform/Personal-VM/Opensuse|openSUSE]] for OS-level details, including system configuration and troubleshooting.

## Hosted Services

### Homepage

Personal homepage and dashboard service providing quick access to home lab tools and services.

- [[Homelab/Platform/Personal-VM/Homepage|Homepage]]

## Service Dependencies

All services run as containers on openSUSE, following the Podman over Docker architecture. The Homepage service serves as the central access point for other home lab services.

## Performance Considerations

The VM is configured with sufficient resources for personal computing tasks, web browsing, and light development work. Resource allocation balances between desktop performance and home lab resource availability.

## Future Expansion

The VM is designed to accommodate additional personal services as needs evolve. Current capacity allows for adding 1-2 additional personal or development-focused services without impacting primary use cases.
