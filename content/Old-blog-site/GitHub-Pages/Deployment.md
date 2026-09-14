---
title: GitHub Pages Deployment
description: How the Hugo blog site is hosted on GitHub Pages, including the worktree approach that was tried and replaced.
tags:
  - GitHub-Pages
  - Hugo
---
# Hosting decision

I hosted the blog's source and the Hugo-generated static site from a dedicated [GitHub account](https://github.com/Mr-Tinkerer), using a repo-specific SSH key, at `mr-tinkerer.github.io`. I chose GitHub for free static hosting alongside the source repo (subject to [GitHub Pages' hosting limits](https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits), e.g. a 1GB size cap).

> [!note] This domain is no longer a blog
> `mr-tinkerer.github.io` used to host the Hugo blog described throughout this bucket. It has since been repurposed to host this very wiki (this Quartz vault) instead — the deployment mechanics below are historical, describing how the *blog* was deployed there, not the wiki's current deployment.

# First attempt: a worktree-based `site` branch, deployed from a branch

My initial plan kept the Hugo project (including `public/`, the generated static output) out of the `main` branch's normal history, using a [[Github-pages-glossary#Git worktree|git worktree]]: `git worktree add --orphan -b site public` creates a `public/` directory backed by a separate, history-less `site` branch. Running `hugo` then updates `public/` for both `main` and `site` simultaneously; I intended to commit each branch independently (a step the [[Publish-Script/Script-Overview|publish script]] was meant to automate).

I then configured GitHub Pages to deploy from a branch, using the `site` branch's root:

![[github-pages-site-broken-after-initial-deploy.png]]
*The published site after my first deploy — CSS is unstyled and links point at unqualified internal paths (e.g. `/posts/Laptop-NAS`) instead of the live GitHub Pages URL.*

Cause: `hugo.toml`'s [[Github-pages-glossary#`baseURL`|`baseURL`]] was left at its default, commented-out placeholder value, so Hugo generated all URLs and asset paths as if the site were local. Setting `baseURL` to the real GitHub Pages URL and rebuilding fixed the *local* build, but pushing that fix did not fix the live site — the deployed `index.html` on GitHub still didn't contain the corrected URL, even though I confirmed the latest commit on the `site` branch was correct and a force-push didn't help either. The `public/` directory shown inside the `main` branch on GitHub also didn't do anything when I clicked it, which pointed at the worktree/branch setup itself, not just the missing `baseURL`.

# Lesson learned: use the official Hugo → GitHub Pages guide instead

I tried to work around this by pointing "deploy from a branch" directly at `public/`, but that wasn't possible — GitHub only offers two folder options for that mode:

![[github-pages-deploy-branch-folder-options-root-or-docs.png]]
*GitHub Pages' "deploy from a branch" mode only allows serving from the branch root or a `/docs` folder — there is no option to point at an arbitrary folder like `public/`.*

[Hugo's own GitHub Pages hosting guide](https://gohugo.io/host-and-deploy/host-on-github-pages/#types-of-sites) recommends [[Github-pages-glossary#GitHub Actions (for Pages)|GitHub Actions]] instead of "deploy from a branch" for exactly this reason. Switching to GitHub Actions meant:

1. Setting the Pages source to "GitHub Actions" in the repo settings, and
2. Adding a workflow file at `.github/workflows/hugo.yaml` that installs Hugo (and Dart Sass, Node.js), runs `hugo build --gc --minify --baseURL "..."`, and uploads/deploys `./public` via `actions/upload-pages-artifact` and `actions/deploy-pages`.

The workflow file is a manually maintained file — I reference it at [hugo.yaml](TODO) in the site repo rather than pasting a copy here, since it's mostly adapted from Hugo's official example and I actively maintain it rather than treating it as a one-off.

![[github-pages-site-fixed-after-actions-deploy.png]]
*The site rendering correctly (styling and links both working) after I switched deployment to GitHub Actions.*

Once this worked, I no longer needed the `site` branch/worktree setup from my first attempt for deployment — GitHub Actions builds directly from `main` and deploys the artifact itself.

# Gotchas

- A `baseURL` left at its placeholder default breaks both absolute links and CSS/asset paths on any host other than `localhost` — I check this first now when a Hugo site looks unstyled or misrouted immediately after publishing.
- "Deploy from a branch" cannot target an arbitrary folder — only the branch root or `/docs`. If the generated site output lives elsewhere (e.g. a worktree-backed `public/`), use GitHub Actions instead of trying to relocate the output to fit the branch-deploy constraint.

# Related

- [[Publish-Script/Script-Overview]] — the script that builds and (previously) would have committed to the `site` worktree branch.
