---
title: "Pt 3, Theme Selection"
description: Evaluating Hugo themes and settling on Archie.
tags:
  - Hugo
---
# Prerequisites

- [[Pt 1, Project-setup-and-page-bundles]]
- [[Pt 2, Fixing-content-rendering]]

---

# Candidates

I shortlisted five themes from the [Hugo themes showcase](https://themes.gohugo.io/): [Terminal](https://github.com/panr/hugo-theme-terminal), [Hugo Blog Awesome](https://github.com/hugo-sid/hugo-blog-awesome), [Diary](https://github.com/AmazingRise/hugo-theme-diary), [Hugo Theme Console](https://github.com/mrmierzejewski/hugo-theme-console), and [Archie](https://github.com/athul/archie).

# Rejected: Hugo Blog Awesome and Diary

Both of these themes failed outright with an RSS build error, rather than just looking wrong, so I dropped them without further testing:

![[hugo-blog-awesome-theme-rss-build-error.png]]
*Hugo Blog Awesome fails to build: its `rss.xml` template can't evaluate `Author` on `page.Site`.*

![[diary-theme-rss-build-error.png]]
*Diary fails the same way: its `index.rss.xml` template also can't evaluate `Author` on `page.Site`.*

# Terminal, Console, and Archie

The remaining three themes (Terminal, Console, Archie) built without errors. All three initially failed to auto-load any of my posts — this turned out to be because they expect posts under `content/posts/`, not directly under `content/`. Moving my posts into `content/posts/` resolved it for all three.

I chose Archie as the final theme: of the three working candidates, it looked the cleanest to me on both desktop and mobile.

# Next

Continue to [[Pt 4, Site-configuration-and-features]].
