---
title: Containerization Tool Choices in Home Lab
description: The real reason behind running a different Linux distro per VM and picking Podman over Docker — deliberately choosing unfamiliar tools for the learning value, not because they're technically superior.
tags:
  - Homelab
  - Containerization
---

## Overview

This page explains the reasoning behind running a different Linux distro per VM and picking Podman over Docker — deliberately choosing unfamiliar tools for the learning value, not because they're technically superior.

## Why These Choices Aren't About "Best Tool for the Job"

The Linux distros were picked because, besides Debian, they hadn't been used before for server hosting. Only Debian, Ubuntu, and Alpine Linux had been used. Each VM was made to run a different, unique distro to learn how to operate new distros. The same underlying reasoning applies to running Podman instead of Docker on every VM: rather than spending home lab setup time learning something unfamiliar, the default would be to reach for what's already known.

## One Distro Per VM

Each Platform VM in this home lab runs a different guest OS, on purpose:

| VM | Guest OS | Notes |
|----|----------|-------|
| Productivity VM | Debian | The one distro of the three actually used before, but paired here with new-to-me services |
| Personal VM | openSUSE | First time running a `firewalld`/SELinux-enforcing distro — see the openSUSE Gotchas page for what that cost |
| Local AI VM | Fedora | First time running an RPM/`dnf`-based distro in this home lab |

See [[Homelab/Logical/Virtual-machines/Vm-allocation|Virtual machines]] for the full VM roster, including the two VMs that don't have services configured yet.

## Podman Over Docker

Podman was picked as the container engine for every VM in this home lab instead of Docker, as another instance of the "learn something new" philosophy. Only Docker had been used before, so running every service on Podman/Quadlet instead was a deliberate choice to learn an unfamiliar tool rather than reach for what's already known. It also holds up well on its own technical merits, which made it an easy choice to commit to across the whole home lab:

- **Daemonless** — Docker relies on a long-running root-owned background daemon (`dockerd`) that everything talks to. Podman has no such daemon — each container is a direct child process of whatever invoked it. That's a smaller attack surface and one less thing that can silently fall over and take every container down with it
- **Rootless-friendly by default** — Podman was built around running containers without root privileges from the start, which matches how services on a shared VM should behave — a compromised container process doesn't automatically mean a compromised root daemon
- **systemd integration via Quadlet** — Since Podman has no daemon to restart containers on boot or on failure the way Docker's daemon does, something else is needed to own that job. Podman Quadlet fills that gap: it turns a declarative `.container`/`.network` unit file into a normal systemd service, so every container gets start-on-boot, restart-on-failure, and dependency ordering (`After=`/`Requires=`) for free, using infrastructure (systemd) that's already on every Linux VM
- **CLI/API compatibility with Docker** — Podman deliberately mirrors the Docker CLI and supports the same image format, so none of the setup knowledge (Dockerfiles, compose-style thinking, image registries) goes to waste — docker-shaped muscle memory just points at Podman instead, and quadlets replace `docker-compose.yml`/`docker run` flags with a small ini-style unit file that systemd understands natively

## Podman Quadlet, in Practice

Each service that uses Podman keeps its quadlet definitions in the infrastructure repo (see that service's own Installation/Setup page for the exact file links) — maintained quadlet files aren't pasted into the wiki directly, per the external-file-reference rule. A quadlet unit typically defines the image, the network, volume mounts, and port/label metadata; systemd then generates and manages the actual service from it, so `systemctl status <name>` works exactly like it would for any other system service.

## Glossary

See the [[Homelab/Logical/Containerization/Containerization-glossary#Podman|Containerization glossary]] for the canonical definitions of Podman and Podman Quadlet.
