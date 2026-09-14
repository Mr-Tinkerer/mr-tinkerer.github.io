---
title: Documentation Rules Overview
description: The rules governing how this vault is organized, and how they came together.
tags:
  - wiki
  - documentation-rules
---

Before migrating any content into this vault, I spent a heavy amount of time working with Claude, ChatGPT, and Gemini (all free versions) to work out the game plan for how the wiki should be organized — what counts as a [[Documentation-rules-glossary#Domain|domain]] vs. a [[Documentation-rules-glossary#Bucket|bucket]], when to split things up, how glossaries work, and so on. That came together as two files:

- [quartz-docs-rules.md](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Wiki/quartz-docs-rules.md) — the main, vault-wide rule set: domain/bucket structure, Hardware/Software/Logical splits, glossaries, sequential workflows, image curation, and the migration process itself.
- [homelab-platform-buckets-rule.md](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Wiki/homelab-platform-buckets-rule.md) — a candidate rule specific to the `Homelab` domain (grouping each VM's guest OS with the services running on it under a `Platform/` bucket, instead of one flat `Software/` bucket). I wasn't sure yet whether this should become a permanent, vault-wide rule or stay a domain-local convention, so it's kept as its own file rather than folded into the main rules.

## Applying them

Every migrated page in this vault — including this domain's own pages — follows `quartz-docs-rules.md`. `homelab-platform-buckets-rule.md` only applies within `Homelab/`.
