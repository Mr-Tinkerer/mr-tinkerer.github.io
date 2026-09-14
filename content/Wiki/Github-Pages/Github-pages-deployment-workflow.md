---
title: Wiki GitHub Pages Deployment Workflow
description: The deploy.yml GitHub Actions workflow that builds and publishes the Quartz site.
tags:
  - wiki
  - github-pages
  - quartz
---

Deployment is handled by a workflow file at `.github/workflows/deploy.yml`, based on the one from [Quartz's own hosting docs](https://quartz.jzhao.xyz/hosting#github-pages): [deploy.yml](https://github.com/Mr-Tinkerer/mr-tinkerer.github.io/blob/v5/.github/workflows/deploy.yaml).

On every push to the `v5` branch, the workflow:

1. Checks out the repo with full git history (Quartz uses git info for page metadata).
2. Sets up Node 24.
3. Caches `npm` dependencies and [[Quartz-glossary#Plugin (Quartz)|Quartz plugins]] between runs.
4. Runs `npm ci`, then `npx quartz plugin install` and `npx quartz build`.
5. Uploads the built `public/` folder as a Pages artifact and deploys it via [[Github-pages-glossary#GitHub Actions (as a Pages source)|GitHub Actions]].

This replaces the old blog's [[Old-blog-site/GitHub-Pages/Deployment|worktree-based branch deploy]] — no separate `site` branch to maintain, GitHub Actions builds and publishes directly from source.
