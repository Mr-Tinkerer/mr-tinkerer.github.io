---
title: Templater Front Matter Template
description: The Obsidian Templater template used to pre-fill a new post's front matter.
tags:
  - Obsidian
  - Automation
---
# What it's for

My new posts need the same front matter shape every time (date, title, slug, description, tags, `toc`) plus a `# Citations` placeholder. Rather than typing this by hand, I use the [Obsidian Templater plugin](https://community.obsidian.md/plugins/templater-obsidian) to insert it automatically for a new note. I wrote the template itself with the help of Claude, rather than hand-learning Templater's templating syntax myself.

For the template to be picked up by the plugin, it has to sit in the vault's configured `Templates` folder — in my case, the vault's root `Templates` folder. I never version-controlled it anywhere; it only ever existed locally in the vault.

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

`<% tp.file.title %>` inserts the note's title automatically; the slug line lowercases the title and replaces spaces with dashes to derive a default slug for me.

# Related

- [[Script-Overview]] — the publish script this front matter feeds into.
