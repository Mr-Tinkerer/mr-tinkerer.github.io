---
title: "Pt 2, Fixing Content Rendering"
description: Getting Obsidian-authored markdown (raw HTML and image embeds) to render correctly under Hugo.
tags:
  - Hugo
---

## Overview

This page documents getting Obsidian-authored markdown (raw HTML and image embeds) to render correctly under Hugo.

## Prerequisites

- [[Pt 1, Project-setup-and-page-bundles]]

---

## Raw HTML is Stripped by Default

Copying an Obsidian post's markdown straight into Hugo's `content` folder mostly worked, but not everything rendered:

See [raw HTML/image not rendering](Images/hugo-raw-html-and-image-not-rendering.png) for the first paragraph of a migrated post rendering in Hugo — the superscript citation marker and the embedded image both fail to appear.

Raw HTML was written for the superscript citation markers (`<sup>[1]</sup>`), and Hugo's default Markdown renderer (Goldmark) strips raw HTML unless told otherwise — a sane default, since it prevents injected HTML from being rendered unexpectedly. Since all of the content here is self-authored, raw HTML was re-enabled in `hugo.toml`:

```toml
[markup]
  [markup.goldmark]
    [markup.goldmark.renderer]
      unsafe = true
```

Setting `unsafe = true` disables Goldmark's raw-HTML sanitization for the whole site — this was accepted as an intentional tradeoff, since every markdown file on the site was written by hand, with no untrusted or third-party content ever passing through the renderer.

See [superscript rendering fixed](Images/hugo-superscript-rendering-fixed-after-unsafe-goldmark.png) for superscript citation markers rendering correctly after enabling `unsafe = true`.

## Images: Obsidian Embeds vs. Hugo/Markdown Links

Obsidian's image embed syntax (`![[Image Name.png]]`) is not standard Markdown. Once posts were reorganized as leaf bundles, the image just needed converting to a standard Markdown image link: `![](Image Name.png)`.

That alone did not work:

See [image link not loading](Images/hugo-image-link-not-loading-space-in-filename.png) for a converted image link still failing to load — the raw markdown syntax shows up as literal text instead of an image.

The cause was spaces in the filename breaking the link. Neither dashes, underscores, nor escaping the spaces with backslashes or quotes fixed it — the fix that worked was to percent-encode spaces as `%20`:

See [image rendering fixed](Images/hugo-image-rendering-fixed-with-percent20-encoding.png) for the same image rendering correctly once spaces in the filename were replaced with `%20`.

This `![]( ... %20 ... )` conversion was automated in the publish script for every post.

---

Next: [[Pt 3, Theme-selection]].
