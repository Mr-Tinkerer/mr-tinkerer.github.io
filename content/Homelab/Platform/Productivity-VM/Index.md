---
title: Productivity-VM Overview
description: Overview of the Productivity VM, its role in the home lab, and the services it hosts.
tags:
  - Homelab
  - Platform
  - Productivity-VM
---

## Overview

This page documents the Productivity VM, its role in the home lab, and the services it hosts.

## Productivity VM Role and Purpose

The Productivity VM provides a dedicated Debian-based platform for productivity-focused services, primarily document management and file sharing. It runs on Proxmox as a LXC container with full system LXC configuration for optimal resource utilization.

## Guest OS

The VM runs Debian as its guest operating system. See [[Homelab/Platform/Productivity-VM/Debian/Debian-overview|Debian Overview]] for OS-level details, including networking configuration, package management, and system maintenance.

## Hosted Services

### Nextcloud

Primary document management and file sharing service with Collabora Office integration for document editing.

- [[Homelab/Platform/Productivity-VM/Nextcloud/Nextcloud-installation|Nextcloud Installation]]

### Stirling-PDF

PDF manipulation tool providing document conversion, merging, splitting, and editing capabilities.

- [[Homelab/Platform/Productivity-VM/Stirling-pdf/Stirling-pdf-installation|Stirling-PDF Installation]]

### ConvertX

E-book conversion service for transforming various ebook formats.

- [[Homelab/Platform/Productivity-VM/Convertx/ConvertX Installation on Productivity VM (Debian)|ConvertX Installation]]

## Service Dependencies

All services run as Podman quadlets on the Debian host, leveraging the Podman over Docker decision for container management. Services share a single Podman network for inter-service communication.

## Performance Considerations

The VM is configured with sufficient CPU and memory resources to handle simultaneous document editing, file serving, and PDF processing. Resource allocation is tuned based on observed usage patterns from the [[Homelab/Logical/Virtual-machines/Vm-allocation|VM allocation roster]].

## Future Expansion

The VM is designed to accommodate additional productivity services as needs evolve. Current capacity allows for 2-3 additional lightweight services without performance degradation.
