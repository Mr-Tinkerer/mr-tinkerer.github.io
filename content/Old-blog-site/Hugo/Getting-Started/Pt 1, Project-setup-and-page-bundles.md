---
title: "Pt 1, Project Setup and Page Bundles"
description: Installing Hugo, creating a project, and organizing posts as leaf bundles so images resolve correctly.
tags:
  - Hugo
---
# Prerequisites

None — this is the first step.

---

# Installing Hugo and creating a project

[Hugo](https://gohugo.io/) is a [[Hugo-glossary#Static site generator|static site generator]]: it takes a folder of content/templates and builds a hostable website. Following Hugo's own [quickstart guide](https://gohugo.io/getting-started/quick-start/), I created the initial project with:

```bash
hugo new project quickstart
cd quickstart
git init
git submodule add https://github.com/gohugo-ananke/ananke themes/ananke
echo "theme = 'ananke'" >> hugo.toml
hugo server
```

This created a new `quickstart` project, initialized a git repo inside it, added the `ananke` theme as a submodule, and started a local dev server at `http://localhost:1313`.

A freshly generated project is blank until content is added — Hugo (or the active theme) automatically surfaces any post placed under `content/` on the home page.

---

# Organizing posts as leaf bundles

Obsidian's image embeds (`![[Some Image.png]]`) are Obsidian-specific — they resolve regardless of where the image sits in the vault, because Obsidian indexes the whole vault. Hugo does not do this; I needed to tell it where an image lives relative to the page that references it.

My fix was Hugo's [[Hugo-glossary#Page bundle|page bundles]]: each post's markdown file and its images live together in the same folder. Since my blog posts don't have nested child posts, I treated each one as a [[Hugo-glossary#Leaf bundle|leaf bundle]] rather than a branch bundle.

![[hugo-blog-publishing-workflow-diagram.png]]
*My overall plan: write in Obsidian, convert to generic markdown, copy the post + images into Hugo's content folder, build with Hugo, then push/reload. I drew this plan before any of the tooling existed, so it isn't reproducible after the fact.*

# Next

Continue to [[Pt 2, Fixing-content-rendering]].
