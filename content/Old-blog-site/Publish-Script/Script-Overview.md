---
title: Publish Script Overview
description: What the Obsidian-to-Hugo publish script does and how it's structured.
tags:
  - Bash
  - Automation
---

## Overview

This page documents what the Obsidian-to-Hugo publish script does and how it's structured.

## What It Does

A Bash script was written that converts Obsidian-authored markdown posts into Hugo-compatible posts and builds the site. It was written by hand (with some AI-assisted regex snippets, noted below), and kept being revised — see [[Script-Improvements]] for the later round of fixes. It's no longer actively maintained in a site repo branch; it's kept, as-is, at [`Archives/blog_publication.sh`](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Archives/blog_publication.sh) in the Project-Dump repo, rather than pasted here.

## Structure

The script runs through these stages:

1. **Setup** — Records a start time (for a duration report at the end) and defines where Obsidian posts live vs. where converted posts should go, creating the destination folder if needed
2. **Select new posts** — Compares the Obsidian posts folder against the existing Hugo posts folder by matching folder basenames, and builds a list of posts that don't have a converted copy yet, so the script only processes new posts instead of the whole site every run
3. **Convert each new post:**
   - Copies the post's folder into the Hugo posts location (preserving the leaf bundle layout)
   - Renames the post's markdown file to `index.md`, since Hugo's leaf bundles require that filename
   - Downgrades every `# ` heading to `## ` so Archie's table of contents (which only reads `##`+) works — see front matter and TOC
   - Rewrites Obsidian image embeds (`![[Image Name.png]]`) into standard Markdown image links with spaces percent-encoded as `%20` — see why this encoding is required
   - Rewrites Obsidian backlinks (`[[File|Alt Text]]`) into standard Markdown links pointing at the target post's slug (see below)
   - Fixes tags so underscores become spaces, since Obsidian's front matter doesn't allow literal spaces in tags but Hugo's does
4. **Build** — Runs `hugo` to generate the static site
5. **Report duration** — Prints how long the run took, in milliseconds

## Resolving Cross-Post Links via a Slug Lookup File

Obsidian backlinks reference posts by filename, but Hugo URLs are built from each post's slug, and posts live under `/posts/`. To bridge this, the script first scans every markdown file in the Obsidian vault and builds a lookup file (`slug.txt`) of `filename|slug: <slug>` pairs, regenerating it from scratch on every run (so slug changes are always picked up). It then finds every Obsidian backlink that contains a `|` (to avoid matching unrelated double-bracket syntax, such as Bash's `[[ ]]` conditionals — see the follow-up fix for this collision), looks up the referenced file's slug, lowercases it, replaces spaces with dashes, and rewrites the link to `/posts/<slug>`.

## AI-Assisted Parts

A few regex/extraction one-liners in the script (extracting the text inside an Obsidian backlink via `grep`'s Perl-regex mode, and extracting front matter/tags via `awk`) were generated with the help of Google Gemini rather than written by hand — this is noted since it couldn't fully derive that regex independently.

## Related

- [[Script-Improvements]] — the follow-up round of fixes (preview, auto-date, error handling, required-field checks)
- [[Templater-Front-Matter-Template]] — the Obsidian template that pre-fills a new post's front matter
- [[GitHub-Pages/Deployment]] — how the built site actually gets published once this script finishes
