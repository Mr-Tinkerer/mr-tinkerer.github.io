---
title: "Pt 4, Site Configuration and Features"
description: Front matter, RSS, menus, table of contents, custom CSS overrides, search, and dark mode on the Archie theme.
tags:
  - Hugo
---
# Prerequisites

- [[Pt 1, Project-setup-and-page-bundles]]
- [[Pt 2, Fixing-content-rendering]]
- [[Pt 3, Theme-selection]]

---

# Front matter

Hugo uses [[Hugo-glossary#Front matter|front matter]] for page metadata. A post's front matter looks like:

```yaml
---
date: 2026-05-20T14:34:00
title: Turning an Old Laptop into a Proxmox Server
slug: Old Laptop Into Proxmox Server
description: I try to turn an old laptop into a Proxmox server, but struggle in the process due to factors out of my control.
tags:
  - Proxmox
  - Laptop
  - Server
  - Fixing
  - Hardware
  - Lots of Issues
---
```

Most fields are self-explanatory except [[Hugo-glossary#Slug|slug]], which sets the last URL path segment explicitly rather than deriving it from the filename.

Archie also reads a `toc: true` field to enable a pinned [[Hugo-glossary#Table of contents (TOC)|table of contents]]. The TOC only picks up `##`-level headings and deeper, so I had to change posts written with a top-level `#` per section to `##` (this heading-level downgrade is one of the things I automated in the [[Publish-Script/Script-Overview|publish script]]).

I use an [Obsidian Templater](https://community.obsidian.md/plugins/templater-obsidian) template to pre-fill this front matter when starting a new post — see [[Publish-Script/Templater-Front-Matter-Template]].

# RSS and menu buttons

Hugo automatically generates an RSS feed at `/index.xml` with no extra configuration. I added menu buttons (for posts, about, search, RSS) via repeated `[[ menu.main ]]` blocks in `hugo.toml`, each with a `name`, `url`, and `weight` (ordering):

```toml
[[ menu.main ]]
  name = "blogs"
  url = "/posts/"
  weight = 1
```

The `about` button requires a corresponding `about.md` at the root of `content/`.

# Custom CSS overrides

Archie's default content width (`800px`) left a lot of unused space next to the pinned TOC. Rather than editing the theme's source, I used the fact that Hugo lets a site override a theme file by placing a same-path file under the site's own `assets/` — see [[Hugo-glossary#Hugo Pipes / assets override|Hugo Pipes / assets override]]. I found the value to change (`--content-max-width` in `:root`) using the browser's inspect-element tool, located it in the theme's `main.css`, copied it to `assets/css/main.css`, and changed it from `800px` to `1110px`.

I used the same override file to force image captions onto their own line:

```css
em {
  display: block;
  margin-bottom: 10px;
}
```

# Search

Archie ships with a built-in search feature. I enabled it by adding a `search.md` page to `content/`:

```yaml
---
title: "Search"
layout: "search"
outputs:
  - html
  - json
---
```

# Dark mode

Archie's `mode` parameter in `hugo.toml` controls the color scheme (`light`, `dark`, `auto`, or `toggle`). I chose `toggle` so visitors can switch themselves via a button in the top right.

# Final site configuration

The finished `hugo.toml` (theme, menu entries, RSS-safe markup config, dark-mode toggle) is a manually maintained file — I keep it at [hugo.toml](TODO) in the site repo rather than pasting a copy here, since the version above only shows individual pieces I added along the way.

# Gotchas

- Goldmark's `unsafe = true` (see [[Pt 2, Fixing-content-rendering]]) trades away Hugo's default HTML-injection protection — I confirmed this was intentional since all site content is self-authored.
- A theme silently failing to list any posts doesn't necessarily mean the theme is broken — I check whether it expects `content/posts/` instead of `content/` before troubleshooting further.
