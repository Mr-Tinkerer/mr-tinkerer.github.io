---
title: Quartz Configuration
description: The quartz.config.yaml changes I made — site title, sidebar plugin layout, and the Phoenix theme.
tags:
  - wiki
  - quartz
---

All of this lives in `quartz.config.yaml`.

## Site title

Set the site's displayed name via `pageTitle`:

```yaml
pageTitle: Mr.Tinkerer's Personal Wiki
```

## Sidebar plugin layout

Each sidebar [[Quartz-glossary#Plugin (Quartz)|plugin]] block controls its own placement:

- `enabled` — turn the plugin on or off.
- `layout.position` — which sidebar it renders in, `left` or `right`.
- `layout.priority` — order within that sidebar. Lower numbers sort to the top, higher numbers sort to the bottom.

```yaml
enabled: true
layout:
  position: right
  priority: 100
```

## Phoenix theme

I switched from Quartz's default theme to the [Phoenix theme](https://quartz-themes.github.io/phoenix/):

```bash
npm install @quartz-themes/core @quartz-themes/phoenix
```

## What actually changed

Four changes from the default `quartz.config.yaml`:

**Site title:**

```yaml
configuration:
  pageTitle: Mr.Tinkerer
  pageTitleSuffix: ""
```

**Theme**, switched to `phoenix` (see [[#Phoenix theme|above]]):

```yaml
- source: "@quartz-themes/core"
    enabled: true
    options:
      theme: phoenix
      mode: both
```

**Table-of-contents plugin**, sidebar priority raised to `30`:

```yaml
- source: "@quartz-community/table-of-contents"
    enabled: true
    order: 50
    layout:
      position: right
      priority: 30
```

**Backlinks plugin**, turned off:

```yaml
- source: "@quartz-community/backlinks"
    enabled: false
    layout:
      position: right
      priority: 50
```

Separately, I also had to manually fix broken backlinks left over from migration — see [[Quartz-gotchas#Broken backlinks after migration|Gotchas]].
