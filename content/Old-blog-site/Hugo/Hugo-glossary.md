---
title: Hugo Glossary
description: Terms used across the Hugo bucket's pages.
tags:
  - Hugo
  - Glossary
---

## Overview

This page documents terms used across the Hugo bucket's pages.

## Static Site Generator

A tool that takes template/content files and generates plain HTML/CSS/JS ahead of time, rather than rendering pages on each request. See [Hugo's own introduction](https://gohugo.io/about/introduction/).

## Page Bundle

A Hugo convention for organizing a page's content by keeping all of that page's files (markdown, images, etc.) together in one folder. See [Hugo's page bundle docs](https://gohugo.io/content-management/page-bundles/).

## Leaf Bundle

A page bundle with no child bundles inside it: just one `index.md` plus whatever images/assets that page needs, all in the same folder. This layout was used for every post on the blog, since each post is a single, self-contained page with nothing nested underneath it — it's also the layout Hugo requires the content file to be named `index.md` for.

## Branch Bundle

A page bundle that can itself contain child bundles, useful for things like a section listing page that groups several leaf bundles under it. This wasn't used anywhere on the blog, since posts don't nest inside other posts, but it's worth knowing about since it's the other half of the bundle system leaf bundles belong to.

## Front Matter

The YAML/TOML block at the top of a content file that stores a page's metadata (title, date, tags, description, etc.). See [Hugo's front matter docs](https://gohugo.io/content-management/front-matter/).

## Goldmark

Hugo's default Markdown rendering engine. By default it strips raw HTML for safety; this is controlled by the `markup.goldmark.renderer.unsafe` config option. See [Hugo's markup configuration docs](https://gohugo.io/configuration/markup/#rendererunsafe).

## Slug

The last URL path segment for a page (e.g. the `old-laptop-into-proxmox-server` in `/posts/old-laptop-into-proxmox-server/`), set explicitly in a post's front matter rather than derived automatically from its file or folder name. This is needed because posts are written and stored under human-readable titles in Obsidian, but Hugo needs a clean, URL-safe path segment to route to; the publish script's slug-lookup step relies on every post declaring one so cross-post links can be rewritten correctly.

## Table of Contents (TOC)

An auto-generated, in-page navigation list built from a page's headings. In the Archie theme this only picks up headings written as `##` (heading level 2) or deeper.

## Hugo Pipes / Assets Override

Hugo's mechanism for shadowing a theme's file (e.g. a CSS file) by placing a same-path file under the site's own `assets/` directory, without editing the theme's source. See [this community writeup on overriding theme CSS](https://discourse.gohugo.io/t/adding-custom-css-to-hugo-modules/55989/2).

## Archie

The Hugo theme this blog site uses; a minimal blog theme with built-in search, RSS, dark-mode toggle, and TOC support. See the [Archie theme repository](https://github.com/athul/archie).
