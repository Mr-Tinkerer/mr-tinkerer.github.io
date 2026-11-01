---
title: "Pt 3, Theme Selection"
description: Evaluating Hugo themes and settling on Archie.
tags:
  - Hugo
---

## Overview

This page documents evaluating Hugo themes and settling on Archie.

## Prerequisites

- [[Pt 1, Project-setup-and-page-bundles]]
- [[Pt 2, Fixing-content-rendering]]

---

## Candidates

Five themes were shortlisted from the Hugo themes showcase: Terminal, Hugo Blog Awesome, Diary, Hugo Theme Console, and Archie.

## Rejected: Hugo Blog Awesome and Diary

Both of these themes failed outright with an RSS build error, rather than just looking wrong, so they were dropped without further testing:

See [Hugo Blog Awesome RSS error](Images/hugo-blog-awesome-theme-rss-build-error.png) for Hugo Blog Awesome failing to build: its `rss.xml` template can't evaluate `Author` on `page.Site`.

See [Diary RSS error](Images/diary-theme-rss-build-error.png) for Diary failing the same way: its `index.rss.xml` template also can't evaluate `Author` on `page.Site`.

## Terminal, Console, and Archie

The remaining three themes (Terminal, Console, Archie) built without errors. All three initially failed to auto-load any posts — this turned out to be because they expected posts under `content/posts/`, not directly under `content/`. Moving posts into `content/posts/` resolved it for all three.

Archie was chosen as the final theme: of the three working candidates, it looked the cleanest on both desktop and mobile.

---

Next: [[Pt 4, Site-configuration-and-features]].
