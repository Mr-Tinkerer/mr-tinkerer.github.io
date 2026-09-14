---
title: Wiki GitHub Pages Repository Setup
description: Retiring the old blog repo and standing up mr-tinkerer.github.io for the wiki.
tags:
  - wiki
  - github-pages
---

Since the wiki replaced the old blog rather than living alongside it, I archived the old blog site's repo and renamed it, then created a fresh [GitHub Pages repo](https://github.com/Mr-Tinkerer/mr-tinkerer.github.io) for the wiki.

## Setting the source to GitHub Actions

Under the repo's **Settings → Pages**, the **Source** dropdown offers two options: **Deploy from a branch** (the classic Pages experience, what the old blog used — see [[Old-blog-site/GitHub-Pages/Deployment|Deployment]]) or **GitHub Actions** (best when a framework needs a build step first). I picked **GitHub Actions**, since Quartz needs to build the site before it can be published.

After saving and adding the [[Github-pages-deployment-workflow|deployment workflow]], the Pages settings confirmed the source and showed the default domain with HTTPS enforced:

![[github-pages-source-set-to-actions.png]]
*GitHub Pages source set to GitHub Actions, saved successfully, serving from `mr-tinkerer.github.io` with HTTPS enforced.*

## Result

Once the workflow ran, the site was live:

![[wiki-site-deployed-homepage.png]]
*The deployed wiki homepage at `mr-tinkerer.github.io`, showing the migrated Explorer sidebar (Daily Driver Laptop, Failed Laptop, Homelab, Old Blog Site) and the Projects index.*
