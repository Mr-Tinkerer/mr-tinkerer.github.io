---
title: "Pt 2, Fixing Content Rendering"
description: Getting Obsidian-authored markdown (raw HTML and image embeds) to render correctly under Hugo.
tags:
  - Hugo
---
# Prerequisites

- [[Pt 1, Project-setup-and-page-bundles]]

---

# Raw HTML is stripped by default

Copying an Obsidian post's markdown straight into Hugo's `content` folder mostly worked, but not everything rendered:

![[hugo-raw-html-and-image-not-rendering.png]]
*The first paragraph of a migrated post rendered in Hugo — the superscript citation marker and the embedded image both fail to appear.*

I had written the superscript citation markers as raw HTML (`<sup>[1]</sup>`), and Hugo's default Markdown renderer ([[Hugo-glossary#Goldmark|Goldmark]]) strips raw HTML unless told otherwise — a sane default, since it prevents injected HTML from being rendered unexpectedly. Since all of the content here is self-authored, I re-enabled raw HTML in `hugo.toml`:

```toml
[markup]
  [markup.goldmark]
    [markup.goldmark.renderer]
      unsafe = true
```

Setting `unsafe = true` disables Goldmark's raw-HTML sanitization for the whole site — I accepted this as an intentional tradeoff, since every markdown file on the site is written by me, with no untrusted or third-party content ever passing through the renderer.

![[hugo-superscript-rendering-fixed-after-unsafe-goldmark.png]]
*Superscript citation markers rendering correctly after enabling `unsafe = true`.*

# Images: Obsidian embeds vs. Hugo/Markdown links

The second issue is that Obsidian's image embed syntax (`![[Image Name.png]]`) is not standard Markdown. Once I reorganized posts as [[Hugo-glossary#Leaf bundle|leaf bundles]] (see [[Pt 1, Project-setup-and-page-bundles]]), the image just needed converting to a standard Markdown image link: `![](Image Name.png)`.

That alone did not work:

![[hugo-image-link-not-loading-space-in-filename.png]]
*A converted image link still fails to load — the raw markdown syntax shows up as literal text instead of an image.*

The cause was spaces in the filename breaking the link. Neither dashes, underscores, nor escaping the spaces with backslashes or quotes fixed it for me — the fix that worked was to percent-encode spaces as `%20`:

![[hugo-image-rendering-fixed-with-percent20-encoding.png]]
*The same image rendering correctly once spaces in the filename are replaced with `%20`.*

This `![]( ... %20 ... )` conversion is exactly what I automated in the [[Publish-Script/Script-Overview|publish script]] for every post.

# Next

Continue to [[Pt 3, Theme-selection]].
