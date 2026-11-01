---
title: "Pt 4, Site Configuration and Features"
description: Front matter, RSS, menus, table of contents, custom CSS overrides, search, and dark mode on the Archie theme.
tags:
  - Hugo
---

## Overview

This page documents front matter, RSS, menus, table of contents, custom CSS overrides, search, and dark mode on the Archie theme.

## Prerequisites

- [[Pt 1, Project-setup-and-page-bundles]]
- [[Pt 2, Fixing-content-rendering]]
- [[Pt 3, Theme-selection]]

---

## Front Matter

Hugo uses front matter for page metadata. A post's front matter looks like:

```yaml
---
date: 2026-05-20T14:34:00
title: Turning an Old Laptop into a Proxmox Server
slug: Old Laptop Into Proxmox Server
description: Try to turn an old laptop into a Proxmox server, but struggle in the process due to factors out of control.
tags:
  - Proxmox
  - Laptop
  - Server
  - Fixing
  - Hardware
  - Lots of Issues
---
```

Most fields are self-explanatory except slug, which sets the last URL path segment explicitly rather than deriving it from the filename.

Archie also reads a `toc: true` field to enable a pinned table of contents. The TOC only picks up `##`-level headings and deeper, so posts written with a top-level `#` per section had to change to `##` (this heading-level downgrade was one of the things automated in the publish script).

An Obsidian Templater template was used to pre-fill this front matter when starting a new post — see [[Publish-Script/Templater-Front-Matter-Template]].

## RSS and Menu Buttons

Hugo automatically generates an RSS feed at `/index.xml` with no extra configuration. Menu buttons (for posts, about, search, RSS) were added via repeated `[[ menu.main ]]` blocks in `hugo.toml`, each with a `name`, `url`, and `weight` (ordering):

```toml
[[ menu.main ]]
  name = "blogs"
  url = "/posts/"
  weight = 1
```

The `about` button requires a corresponding `about.md` at the root of `content/`.

## Custom CSS Overrides

Archie's default content width (`800px`) left a lot of unused space next to the pinned TOC. Rather than editing the theme's source, Hugo's ability to let a site override a theme file by placing a same-path file under the site's own `assets/` was used. The value to change (`--content-max-width` in `:root`) was found using the browser's inspect-element tool, located in the theme's `main.css`, copied to `assets/css/main.css`, and changed from `800px` to `1110px`.

The same override file was used to force image captions onto their own line:

```css
em {
  display: block;
  margin-bottom: 10px;
}
```

## Search

Archie ships with a built-in search feature. It was enabled by adding a `search.md` page to `content/`:

```yaml
---
title: "Search"
layout: "search"
outputs:
  - html
  - json
---
```

## Dark Mode

Archie's `mode` parameter in `hugo.toml` controls the color scheme (`light`, `dark`, `auto`, or `toggle`). `toggle` was chosen so visitors can switch themselves via a button in the top right.

## Final Site Configuration

The finished `hugo.toml` (theme, menu entries, RSS-safe markup config, dark-mode toggle) is a manually maintained file — it's kept in the site repo rather than pasted here, since the version above only shows individual pieces added along the way.

## Gotchas

- Goldmark's `unsafe = true` trades away Hugo's default HTML-injection protection — this was confirmed intentional since all site content is self-authored
- A theme silently failing to list any posts doesn't necessarily mean the theme is broken — whether it expects `content/posts/` instead of `content/` is checked before troubleshooting further
