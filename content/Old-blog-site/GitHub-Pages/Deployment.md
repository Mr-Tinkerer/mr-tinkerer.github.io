---
title: GitHub Pages Deployment
description: How the Hugo blog site is hosted on GitHub Pages, including the worktree approach that was tried and replaced.
tags:
  - GitHub-Pages
  - Hugo
---

## Overview

This page documents how the Hugo blog site is hosted on GitHub Pages, including the worktree approach that was tried and replaced.

## Hosting Decision

The blog's source and the Hugo-generated static site were hosted from a dedicated GitHub account, using a repo-specific SSH key, at `mr-tinkerer.github.io`. GitHub was chosen for free static hosting alongside the source repo (subject to GitHub Pages' hosting limits, e.g. a 1GB size cap).

> [!note] This domain is no longer a blog
> `mr-tinkerer.github.io` used to host the Hugo blog described throughout this bucket. It has since been repurposed to host this very wiki (this Quartz vault) instead — the deployment mechanics below are historical, describing how the blog was deployed there, not the wiki's current deployment. The old repo is archived for reference purposes.

## First Attempt: A Worktree-Based `site` Branch, Deployed from a Branch

The initial plan kept the Hugo project (including `public/`, the generated static output) out of the `main` branch's normal history, using a git worktree: `git worktree add --orphan -b site public` creates a `public/` directory backed by a separate, history-less `site` branch. Running `hugo` then updates `public/` for both `main` and `site` simultaneously; the intention was to commit each branch independently (a step the publish script was meant to automate).

GitHub Pages was configured to deploy from a branch, using the `site` branch's root:

See [broken site after initial deploy](Images/github-pages-site-broken-after-initial-deploy.png) for the published site after the first deploy — CSS is unstyled and links point at unqualified internal paths (e.g. `/posts/Laptop-NAS`) instead of the live GitHub Pages URL.

**Cause:** `hugo.toml`'s `baseURL` was left at its default, commented-out placeholder value, so Hugo generated all URLs and asset paths as if the site were local. Setting `baseURL` to the real GitHub Pages URL and rebuilding fixed the *local* build, but pushing that fix did not fix the live site — the deployed `index.html` on GitHub still didn't contain the corrected URL, even though the latest commit on the `site` branch was correct and a force-push didn't help either. The `public/` directory shown inside the `main` branch on GitHub also didn't do anything when clicked, which pointed at the worktree/branch setup itself, not just the missing `baseURL`.

## Lesson Learned: Use the Official Hugo → GitHub Pages Guide Instead

An attempt to work around this by pointing "deploy from a branch" directly at `public/` failed — GitHub only offers two folder options for that mode:

See [deploy branch folder options](Images/github-pages-deploy-branch-folder-options-root-or-docs.png) for GitHub Pages' "deploy from a branch" mode only allowing serving from the branch root or a `/docs` folder — there is no option to point at an arbitrary folder like `public/`.

[Hugo's own GitHub Pages hosting guide](https://gohugo.io/host-and-deploy/host-on-github-pages/#types-of-sites) recommends GitHub Actions instead of "deploy from a branch" for exactly this reason. Switching to GitHub Actions meant:

1. Setting the Pages source to "GitHub Actions" in the repo settings, and
2. Adding a workflow file at `.github/workflows/hugo.yaml` that installs Hugo (and Dart Sass, Node.js), runs `hugo build --gc --minify --baseURL "..."`, and uploads/deploys `./public` via `actions/upload-pages-artifact` and `actions/deploy-pages`.

The workflow file is a manually maintained file — it's referenced at hugo.yaml in the site repo rather than pasted here, since it's mostly adapted from Hugo's official example and is actively maintained rather than treated as a one-off.

See [fixed site after Actions deploy](Images/github-pages-site-fixed-after-actions-deploy.png) for the site rendering correctly (styling and links both working) after switching deployment to GitHub Actions.

Once this worked, the `site` branch/worktree setup from the first attempt was no longer needed for deployment — GitHub Actions builds directly from `main` and deploys the artifact itself.

## Gotchas

- A `baseURL` left at its placeholder default breaks both absolute links and CSS/asset paths on any host other than `localhost` — this is checked first now when a Hugo site looks unstyled or misrouted immediately after publishing
- "Deploy from a branch" cannot target an arbitrary folder — only the branch root or `/docs`. If the generated site output lives elsewhere (e.g. a worktree-backed `public/`), GitHub Actions is used instead of trying to relocate the output to fit the branch-deploy constraint

## Related

- [[Publish-Script/Script-Overview]] — the script that builds and (previously) would have committed to the `site` worktree branch
