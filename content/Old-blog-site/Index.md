---
title: Old Blog Site
description: The tooling behind the old Hugo-based blog site, formerly hosted at mr-tinkerer.github.io (now repurposed to host this wiki) — building it, publishing posts to it, and hosting it.
tags:
  - Hugo
  - GitHub-Pages
---
This domain covers how I built and maintained my old Hugo-based blog site (formerly at mr-tinkerer.github.io, which now hosts this wiki instead) — the publishing pipeline, not the subject matter of the posts I published through it. It spans three buckets: the [[#Hugo|Hugo]] static site itself, the [[#Publish-Script|custom Bash publish script]] I wrote to convert Obsidian posts into Hugo posts, and [[#GitHub-Pages|GitHub Pages]] hosting/deployment.

# Hugo

How I set up the Hugo project, got my Obsidian-authored markdown to render correctly, chose a theme, and configured site features (front matter, RSS, menus, table of contents, custom CSS, search, dark mode).

- [[Hugo/Hugo-glossary|Glossary]]
- [[Hugo/Getting-Started/Pt 1, Project-setup-and-page-bundles|Pt 1, Project Setup and Page Bundles]]
- [[Hugo/Getting-Started/Pt 2, Fixing-content-rendering|Pt 2, Fixing Content Rendering]]
- [[Hugo/Getting-Started/Pt 3, Theme-selection|Pt 3, Theme Selection]]
- [[Hugo/Getting-Started/Pt 4, Site-configuration-and-features|Pt 4, Site Configuration and Features]]

# Publish-Script

The custom Bash script I wrote to convert Obsidian markdown posts into Hugo-ready posts (image/backlink/tag fixes, slug lookups) and build the site, plus the later revisions I made (preview, auto-dating, error handling, required-field checks) and the Templater template I use to seed a new post's front matter.

- [[Publish-Script/Publish-script-glossary|Glossary]]
- [[Publish-Script/Script-Overview|Script Overview]]
- [[Publish-Script/Script-Improvements|Script Improvements]]
- [[Publish-Script/Templater-Front-Matter-Template|Templater Front Matter Template]]

# GitHub-Pages

How I hosted the built site on GitHub Pages: the account/repo setup, the worktree-based approach I tried and abandoned, and the switch to a GitHub Actions-based deployment that actually worked.

- [[GitHub-Pages/Github-pages-glossary|Glossary]]
- [[GitHub-Pages/Deployment|Deployment]]
