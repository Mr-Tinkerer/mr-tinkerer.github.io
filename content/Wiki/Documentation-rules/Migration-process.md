---
title: Migration Process
description: Using Claude Code to migrate the old blog content into the wiki per the documentation rules.
tags:
  - wiki
  - documentation-rules
---

With [[Docs-rules-overview|the rules]] written, I used Claude Code (via the Anthropic API, not a subscription plan) to actually carry out the migration — pointing it at the old blog content and the rules files and having it restructure everything into this vault's [[Documentation-rules-glossary#Domain|domain]]/[[Documentation-rules-glossary#Bucket|bucket]] layout.

## Lessons learned

The session ran long — a single 19-hour-plus Claude Code session against ~1.1GB of source material:

![[claude-code-migration-session-stats.png]]
*Claude Code session stats for the migration: 189.6m tokens across a single session, longest session 19h 10m 16s — driven by the size of the source material below, not by the number of topics being migrated.*

At first I chalked this up to just not directing the migration efficiently, but the real cause was the source material itself: 1.1GB of it, made up of 582 image files. That bulk is what actually ballooned the token count — not inefficient prompting, and not the scope of what got migrated.

Where the actual waste was: I'd gotten lazy with the skeleton drafts and just screenshotted everything as I went instead of writing proper notes. Every one of those screenshots then had to get read and reasoned about during migration, which is what drove the size and length of the session.

To be clear about what wasn't wasted:
- The unwritten blog post skeletons weren't wasted migrating — every one of them was actually planned content I intend to write, not abandoned drafts.
- The other saved notes weren't wasted either — same reasoning.

So the fix for next time isn't "migrate less" — it's writing proper notes as I go instead of defaulting to a screenshot, so the source material migration actually has to work from is smaller and more deliberate.

The migration also introduced broken [[Documentation-rules-glossary#Wikilink|wikilinks]] that I had to fix afterward — see [[Quartz-gotchas#Broken backlinks after migration|Quartz Gotchas]].
