---
title: GitHub-Pages Glossary
description: Terms used across the GitHub-Pages bucket's pages.
tags:
  - GitHub-Pages
  - Glossary
---
# Glossary

## GitHub Pages — GitHub's free static-site hosting for a repository, with two source modes: "deploy from a branch" (serves a chosen branch/folder as-is) or GitHub Actions (a workflow builds the site and uploads the result as a deployable artifact). See [GitHub's Pages documentation](https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits). Pages sites have hosting limits (e.g. a 1GB size cap).

## Git worktree — A Git feature that checks out an additional branch into its own working directory while sharing the same underlying repository, via `git worktree add`. Using `--orphan -b <branch>` creates the new branch with no shared history with the branch it was created from. See [background on worktree-based workflows](https://nameocean.net/article/supercharge-your-git-workflow-how-worktree-based-team-coordination-transforms-collaborative-development/).

## `baseURL` — The Hugo config value (`hugo.toml`) that Hugo uses as the root when generating absolute URLs and asset paths (CSS, JS, images) throughout a built site. It defaults to a placeholder like `http://example.org/`, which works fine on `localhost` during local development but breaks the moment the site is deployed anywhere else, since every generated link and asset path still points at the placeholder host instead of the real one. I hit this directly on my first GitHub Pages deploy (see [[Deployment]]) — the fix is setting `baseURL` to the actual live URL (e.g. `https://mr-tinkerer.github.io/`) before building.

## GitHub Actions (for Pages) — A workflow-based deployment mode where a `.github/workflows/*.yaml` file defines steps to build the site (installing Hugo, running `hugo build`) and upload the result via `actions/upload-pages-artifact` + `actions/deploy-pages`, instead of serving a branch's raw files directly. See [Hugo's official GitHub Pages hosting guide](https://gohugo.io/host-and-deploy/host-on-github-pages/#types-of-sites).
