---
title: Templater Front Matter Template
description: The Obsidian Templater template used to pre-fill a new post's front matter.
tags:
  - Obsidian
  - Automation
---

## Overview

This page documents the Obsidian Templater template used to pre-fill a new post's front matter.

## What It's For

New posts need the same front matter shape every time (date, title, slug, description, tags, `toc`) plus a `# Citations` placeholder. Rather than typing this by hand, the Obsidian Templater plugin is used to insert it automatically for a new note. The template itself was written with Claude's help, rather than hand-learning Templater's templating syntax.

For the template to be picked up by the plugin, it sits in the vault's configured `Templates` folder — in this case, the vault's root `Templates` folder. It was never version-controlled anywhere; it only existed locally in the vault.

The template's contents (shown here for reference, not as a maintained/versioned copy):

```markdown
---
date: 
title: <% tp.file.title %>
slug: <% tp.file.title.toLowerCase().replace(/\s+/g, '-') %>
description:
tags:
toc: true
---
# The 

# Citations
> [1] 
```

`<% tp.file.title %>` inserts the note's title automatically; the slug line lowercases the title and replaces spaces with dashes to derive a default slug.

## Related

- [[Script-Overview]] — the publish script this front matter feeds into
