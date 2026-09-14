---
title: Containerization Glossary
description: Terms used across the Containerization bucket.
tags:
  - Homelab
  - glossary
---
## Podman

A daemonless container engine, API-compatible with Docker but without a long-running background daemon. It's the container engine I run every containerized service on across this home lab — see [[Homelab/Logical/Containerization/Tool-choices|Tool-choice philosophy]] for why I picked it over Docker. See [Podman (official site)](https://podman.io/).

## Podman Quadlet

A systemd generator that turns a declarative `.container`/`.network` unit file into a managed systemd service, so containers behave like normal system services (start on boot, restart policies, dependency ordering) despite Podman being daemonless. See [Quadlet (Podman docs)](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html).
