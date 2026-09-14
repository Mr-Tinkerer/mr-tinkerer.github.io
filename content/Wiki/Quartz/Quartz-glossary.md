---
title: Quartz Glossary
description: Terms used across the Quartz bucket.
tags:
  - wiki
  - quartz
  - glossary
---

## Quartz

A [[Old-blog-site/Hugo/Hugo-glossary#Static site generator|static site generator]] built specifically to publish [[#Obsidian vault|Obsidian vaults]] as browsable websites, preserving wikilinks, backlinks, and graph views. It's what turns this vault into the site you're reading. See [Quartz's own docs](https://quartz.jzhao.xyz/) for the full reference.

![[Old-blog-site/Hugo/Hugo-glossary#Static site generator]]

## Obsidian vault

A folder of Markdown files (plus attachments) managed by the [Obsidian](https://obsidian.md/) note-taking app, which treats `[[wikilinks]]` between files as first-class relationships and can render a graph of how notes connect. This wiki is an Obsidian vault; Quartz reads it directly and republishes those same wikilinks as site navigation and backlinks.

## Plugin (Quartz)

A modular unit of Quartz functionality — a transformer, filter, or emitter — that can be enabled, disabled, and reordered independently in `quartz.config.yaml`. Sidebar components (search, backlinks, table of contents, themes) are all plugins, each with its own `layout` block controlling which side of the page it renders on and in what order.
