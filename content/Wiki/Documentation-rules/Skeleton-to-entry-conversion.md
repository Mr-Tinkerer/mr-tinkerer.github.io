---
title: Skeleton-to-Entry Conversion Workflow
description: The generic process for turning a skeleton note into a wiki entry with AI help, and the checklist to follow next time.
tags:
  - wiki
  - documentation-rules
---

This is a summary of how I actually convert a [[Documentation-rules-glossary#Skeleton|skeleton]] into a proper wiki entry in conversation with an AI assistant, distinct from [[Migration-process|the one-off Claude Code migration]] that moved the old blog wholesale. That was a bulk, largely unsupervised pass; this is the back-and-forth process for one skeleton at a time, and it's the one I expect to reuse going forward.

## The process

1. **Hand over the skeleton, plus context the assistant can't infer.** The raw skeleton note, any images it references, and — critically — a dump of the vault's current structure and headings, so the assistant can find existing buckets and terms to link against instead of guessing or duplicating them. Two commands cover this:

   Folder tree, skipping the shared image folders:
   ```bash
   tree -I "Images"
   ```

   Every heading in every file, so the assistant can spot existing pages/terms without opening each one:
   ```bash
   find . -type f -name "*.md" -print0 | while IFS= read -r -d '' file; do
       echo -n "Heading found in $file: "
       grep "^#" "$file"
       echo "------------------"
   done
   ```
2. **Identify the domain and bucket(s).** Does the topic belong to an existing domain, or is it a new top-level project? Within that domain, does the content split across multiple buckets (one tool/service/OS per bucket), or is it one bucket? Check whether the domain crosses the Hardware/Software/Logical split threshold (§3) before assuming a grouped structure — a domain with no hardware entries stays flat.
3. **Curate images deliberately, not by default.** For each image in the skeleton, ask: does it diagnose a problem, verify a fix, record a non-reproducible fact, or document a real constraint? If it's just narrating a step-by-step UI action (e.g. an installer's language-selection screen) or an incidental screenshot, describe the step in text instead and drop the image. Keep the ones that carry real evidence — a hardware spec sheet that ruled out an option, a permission-denied error that pinned down where a fault actually was, a stats screenshot that's the only record of a one-time result. Don't assume a screenshot is worth keeping just because it was captured, and don't assume it's worth discarding just because it looks like narration either — check with the reader/source author when it's ambiguous rather than guessing either way.
4. **Never paste a user-maintained file wholesale.** Link to it on GitHub instead (§17). If pasting a diff or a hand-extracted snippet to illustrate a change, make sure it's genuinely illustrative — not a stand-in for the full file — and confirm the extraction is accurate rather than assuming a diff or summary is good enough on its own. The same goes for scripts and systemd units referenced in a skeleton: link to where they're maintained rather than reproducing them in the entry.
5. **Pull out glossary terms as they come up**, checking first whether the term already has a canonical definition elsewhere in the vault. Transclude the canonical entry instead of redefining it.
6. **Rewrite the voice, don't just reformat it.** A skeleton/blog post is written in the moment, first-person and narrated ("you might be wondering why...", "I've decided to continue anyway"). The wiki entry keeps the first person but drops the narration and blogging asides — state what was done and why, not the live thought process of deciding it. Keep the reasoning, lose the color commentary.
7. **Keep AI-assistance attribution specific.** If a skeleton says a command, a diagnosis, or a fix came from asking an AI tool (Claude, Gemini, ChatGPT, Claude Code, etc.), preserve that attribution in the entry rather than presenting the result as if it were worked out unassisted — including cases where the assistant doing the conversion is itself the one that did the original work.
8. **Flag anything you can't answer instead of guessing.** Missing URLs, unclear methods, ambiguous causal claims — write these into `Migration Q&A.json` as open questions rather than inventing an answer. Resolve them into the actual pages once answered, and delete the file once nothing's left open.
9. **Update the parent index.** A new domain needs a link from the vault root `Index.md`; a new bucket needs a link from its domain's `Index.md`.
10. **Expect a correction pass.** The first draft will get some emphasis wrong — what actually mattered, which detail was the real cause vs. incidental, which extraction method to use for a snippet. Treat the source author's corrections as the source of truth over your own inference, and update every page the wrong claim propagated to, not just the one it was raised on.

## What to watch for

- **Don't guess intent from an image alone.** A screenshot can look like idle narration and still be the only evidence for the real problem — confirm before discarding something that might be diagnostic.
- **Don't infer causality that wasn't stated.** A config change and a bug fix happening around the same time doesn't mean one caused the other.
- **Keep whoever's role visible.** If an AI assistant did migration work, drafted rules, fixed something in the vault, or supplied a command/diagnosis in the original source material, say so in the entry rather than writing it as if it happened unassisted.
- **A skeleton's own structure isn't the entry's structure.** A blog post's section breaks follow how the story unfolded, not how the content should be organized into buckets/pages — expect to regroup, not just retitle headings.
