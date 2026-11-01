---
title: Old Blog Site
description: The tooling behind the old Hugo-based blog site, formerly hosted at mr-tinkerer.github.io (now repurposed to host this wiki) — building it, publishing posts to it, and hosting it.
tags:
  - Hugo
  - GitHub-Pages
---

## Overview

This domain covers how the old Hugo-based blog site was built and maintained (formerly at mr-tinkerer.github.io, which now hosts this wiki instead) — the publishing pipeline, not the subject matter of the posts published through it. It spans three buckets: the Hugo static site itself, the custom Bash publish script written to convert Obsidian posts into Hugo posts, and GitHub Pages hosting/deployment.

## Hugo

How the Hugo project was set up, Obsidian-authored markdown was got to render correctly, a theme was chosen, and site features were configured (front matter, RSS, menus, table of contents, custom CSS, search, dark mode).

- [[Hugo/Hugo-glossary|Glossary]]
- [[Hugo/Getting-Started/Pt 1, Project-setup-and-page-bundles|Pt 1, Project Setup and Page Bundles]]
- [[Hugo/Getting-Started/Pt 2, Fixing-content-rendering|Pt 2, Fixing Content Rendering]]
- [[Hugo/Getting-Started/Pt 3, Theme-selection|Pt 3, Theme Selection]]
- [[Hugo/Getting-Started/Pt 4, Site-configuration-and-features|Pt 4, Site Configuration and Features]]

## Publish-Script

The custom Bash script written to convert Obsidian markdown posts into Hugo-ready posts (image/backlink/tag fixes, slug lookups) and build the site, plus the later revisions made (preview, auto-dating, error handling, required-field checks) and the Templater template used to seed a new post's front matter.

- [[Publish-Script/Publish-script-glossary|Glossary]]
- [[Publish-Script/Script-Overview|Script Overview]]
- [[Publish-Script/Script-Improvements|Script Improvements]]
- [[Publish-Script/Templater-Front-Matter-Template|Templater Front Matter Template]]

## GitHub Pages

How the built site was hosted on GitHub Pages: the account/repo setup, the worktree-based approach tried and abandoned, and the switch to a GitHub Actions-based deployment that actually worked.

- [[GitHub-Pages/Github-pages-glossary|Glossary]]
- [[GitHub-Pages/Deployment|Deployment]]
