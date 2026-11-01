---
title: Containerization Glossary
description: Terms used across the Containerization bucket.
tags:
  - Homelab
  - glossary
---

## Overview

This page documents terms used across the Containerization bucket.

## Podman

A daemonless container engine, API-compatible with Docker but without a long-running background daemon. It's the container engine run for every containerized service across this home lab. See the Tool-choice philosophy page for why Podman was picked over Docker.

See [Podman (official site)](https://podman.io/).

## Podman Quadlet

A systemd generator that turns a declarative `.container`/`.network` unit file into a managed systemd service, so containers behave like normal system services (start on boot, restart policies, dependency ordering) despite Podman being daemonless.

See [Quadlet (Podman docs)](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html).
