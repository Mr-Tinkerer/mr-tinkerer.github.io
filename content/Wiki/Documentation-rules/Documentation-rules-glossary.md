---
title: Documentation Rules Glossary
description: Vault-specific vocabulary used by the migration rules.
tags:
  - wiki
  - documentation-rules
  - glossary
---

## Domain

A top-level folder in this vault representing a broad project or context — `Homelab/`, `Daily-driver-laptop/`, `Old-blog-site/`, `Wiki/`. Each domain has exactly one `Index.md` and organizes its own buckets underneath it; domains don't share buckets with each other.

## Bucket

A folder within a [[#Domain|domain]] representing one concrete item, OS, service, or tool (or, for a Logical bucket, one cross-cutting convention). A bucket holds that subject's own pages and exactly one `Glossary.md`; it does not get its own `Index.md`.

## Wikilink

An Obsidian-style `[[Page-name]]` link between vault files. Quartz renders these as normal hyperlinks and also uses them to generate backlinks and the site's graph view.

## Transclusion

A wikilink prefixed with `!`, e.g. `![[Page-name#Heading]]`, which embeds another page's (or section's) content directly rather than just linking to it. Used to reuse a canonical glossary definition across buckets without copying the text.

## Migration Q&A file

`Migration Q&A.json` at the vault root — a running list of unresolved clarification questions raised while migrating content, each with an ID, type, the question itself, and context. Deleted once no questions remain. See [[Migration-process|Migration Process]].

## Entry Q&A file

`Entry Q&A.json` at the vault root — the running list of unresolved clarification questions raised while adding new entries after the migration. It uses the same structure as the [[#Migration Q&A file|Migration Q&A file]] (ID, type, question, context, related pages, and an answer field I fill in). It is also where flagged concerns go, such as a possible security mistake in the source material. Answered questions get folded into the affected pages and removed, and the file is deleted once no questions remain. See [[Docs-rules-overview|Documentation Rules Overview]].

## Skeleton

A draft note in the vault — an outline, a dump of screenshots, or a half-written page — that hasn't yet been converted into a proper wiki entry following this vault's rules. See [[Skeleton-to-entry-conversion|Skeleton-to-Entry Conversion Workflow]].
