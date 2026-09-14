---
title: Quartz Gotchas
description: Problems I hit getting the site to render correctly after migration.
tags:
  - wiki
  - quartz
---

## SVGs don't render

`.svg` image embeds weren't working in the published site. Fix: replaced the affected images with `.png` versions instead of chasing the SVG rendering issue.

## Broken backlinks after migration

Claude Code introduced broken backlinks while migrating content into the vault (see [[Migration-process|Migration Process]]). Fix: told Claude to go back through the vault and fix any broken links it found, rather than fixing them by hand.
