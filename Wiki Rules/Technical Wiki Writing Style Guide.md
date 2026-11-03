# Technical Wiki Writing Style Guide

This guide is a complete, self-contained specification for rewriting technical wiki entries. An LLM given this guide and a raw wiki entry must be able to produce a rewritten entry that matches the style defined here without any additional context, examples, or reference material.

Read the entire guide before rewriting an entry. The final checklist at the end must be satisfied for every entry.

**Input:** A raw wiki entry that may be written in a narrative, first-person, or AI-generated style.
**Output:** A rewritten entry that follows every rule in this guide.

---

## 1. Core Philosophy: Blended Authority

The target style is a blend of two reference styles. The blend is not a compromise between them; it combines the strongest element of each.

### 1.1 Arch Wiki precision

From the Arch Wiki, take:

- **Declarative, encyclopedic tone.** State facts and procedures directly. Do not narrate the process of discovering them.
- **Technical density.** Every sentence should carry technical information. Remove sentences that only provide pacing or transition.
- **Precise terminology.** Use the exact names of daemons, packages, configuration parameters, error codes, and protocols. Do not paraphrase technical identifiers.
- **Assumed competence.** Write for a reader who understands the domain. Do not over-explain basic concepts unless the entry is explicitly introductory.
- **Short paragraphs.** Two to four sentences per paragraph. One idea per paragraph.

### 1.2 Minecraft Wiki explanatory structure

From the Minecraft Wiki, take:

- **Labeled sub-points within sections.** Break a section into bold lead-in labels followed by one to three sentences. For example: `**Error State:** The system consistently reports a kernel-level error (`23=IEEE8021X_FAILED`).` Each label isolates one distinct fact or mechanism.
- **Mechanism over symptom.** Explain why a system behaves the way it does, not just what the behavior looks like.
- **Explicit isolation.** When describing a problem, state what has been ruled out and how. For example: "Testing connectivity against a standard WPA2-PSK network confirms that the Wi-Fi hardware and driver stack are operational, isolating the issue exclusively to the enterprise authentication workflow."
- **Defined terms.** When introducing a term that may be unfamiliar, define it in the same sentence or the next sentence.

### 1.3 What the blend sounds like

The blend produces sentences that are technically precise and structurally clear. A typical paragraph:

> The `wpa_supplicant` daemon requires the bare, canonical method identifier, `MSCHAPV2`. The inclusion of the `EAP-` prefix invalidates the configuration from the software's perspective, causing the authentication stage to fail silently.

This is dense (two sentences, four technical terms) but clear (each sentence states one fact and its consequence). It does not narrate discovery, does not use first person, does not use metaphor, and does not use filler.

---

## 2. Absolute Prohibitions

These rules have no exceptions. Violating any of them invalidates the entry.

### 2.1 No emojis

No emojis anywhere in the document. Not in the title, not in headings, not in body text, not in callouts, not in lists, not in table cells. This applies to every character in the Unicode emoji ranges. If a heading needs emphasis, use a bold label or a callout, not an emoji.

This is the single most frequently violated rule. Check for it explicitly before returning any output.

### 2.2 No em dashes

Do not use the em dash character (—). Do not use the en dash (–) as a substitute. Connect ideas with commas, periods, semicolons, or colons. Use a colon to introduce a list or a definition.

- Wrong: "The router was replaced — the old one had no Ethernet port."
- Right: "The router was replaced because the old one had no Ethernet port."

### 2.3 No first-person narration

Do not use first-person pronouns or verbs: I, me, my, we, our, us. Do not write "I found," "I spent," "I set," "I then," "my router," "we tested."

Do not replace first person with second person ("you will find," "your router"). Use imperative mood for procedures ("Set the value," "Run the command") and declarative mood for descriptions ("The system fails," "The daemon requires").

### 2.4 No narrative of discovery

Do not describe the process of solving a problem. Do not mention how long a problem took, what was searched online, which tools or people were consulted, or what was tried and abandoned unless the abandoned attempt is technically relevant to the final solution.

- Bad: "I spent about an hour searching online, then six hours working through it with Claude before landing on the root cause."
- Good: "The root cause is a mismatch between the value written by the configuration generator and the value required by the daemon."

If an abandoned attempt is technically relevant (for example, a boot medium that failed for a specific reason), describe the failure and its cause, not the experience of encountering it.

### 2.5 No metaphors, similes, or clichés

Describe technical behavior in technical terms. Do not compare system components to everyday objects. Do not use phrases like "under the hood," "the heart of the system," "a double-edged sword," or "tip of the iceberg."

### 2.6 No "not just X, but also Y" constructions

Do not use the pattern "not just X, but also Y" or any variant ("not only X, but Y as well," "X is more than just Y," "it's not merely X, it's Y"). State each fact separately.

### 2.7 No concluding or transitional filler

Do not use phrases such as:

- in conclusion
- in closing
- to summarize
- to sum up
- it is important to note
- it is worth mentioning
- as mentioned earlier
- as we have seen
- let's dive in
- without further ado

State the information directly. If a transition is needed, use a period or a semicolon.

### 2.8 No source footers

Do not append a line crediting a source, model, or adaptation at the end of the document. Do not write "Source: ..." or "Adapted from ..." as a footer.

---

## 3. Document Structure

### 3.1 Standalone troubleshooting entry

Use this structure when the entry documents a problem and its fix:

```
# Title

## Overview
One to three sentences stating what the entry covers and what problem it addresses.

## Observed Issue
The symptom, the exact error, and what has been ruled out.

## Root Cause Analysis
The precise technical mismatch, broken into labeled points.

## Solution & Implementation Steps
Numbered imperative steps with code blocks.

## Verification
How to confirm the fix worked.

## Caveats (optional)
Critical warnings about limitations or side effects, in a callout.
```

### 3.2 Step-by-step series entry

Use this structure when the entry is part of a numbered sequence:

```
# Title

## Prerequisites
A bulleted list of prior entries in the series and any required tools or materials.

## Overview
One to three sentences stating what the entry covers.

## Body sections
One or more H2 sections covering the procedure or concept.

## Next Step
A link to the next entry in the series.
```

### 3.3 Prerequisites placement

- If the prerequisites are prior steps in a series, place the Prerequisites section before the Overview.
- If the prerequisites are tools, materials, or access requirements, place the Prerequisites section after the Overview.

### 3.4 Heading hierarchy

- H1: the entry title. One per document.
- H2: major sections (Overview, Observed Issue, Root Cause Analysis, Solution, Verification, Prerequisites, Next Step).
- H3: sub-sections within a major section, used only when a section covers more than one distinct topic.

Do not skip levels (do not go from H1 to H3). Do not use H4 or deeper unless the content genuinely requires four levels.

---

## 4. Section-by-Section Requirements

### 4.1 Overview

The Overview states the scope of the entry in one to three sentences. It answers: what does this entry cover, and what problem or task does it address?

- Do not begin with "This article" or "This page." Begin with the subject or action.
- Do not include background, history, or motivation.
- Do not include a summary of the solution.

Example:

> This guide details the process of successfully connecting an OpenWrt device to a WPA2-EAP (PEAP / MSCHAPv2) enterprise network, addressing a known bug within the system's configuration generator.

### 4.2 Observed Issue

The Observed Issue section describes what goes wrong. It contains three parts, each introduced by a bold label:

- **A description of the failure.** One to three sentences stating what the system does when the problem occurs.
- **Error State:** The exact error message, code, or log line. Use inline code formatting.
- **Isolation:** What has been ruled out and how. State the test that was performed and the result that narrows the problem.

Example:

> The system fails to establish a connection when targeting a WPA2-EAP (PEAP/MSCHAPv2) enterprise network. While the installation of the `wpa_supplicant` package is necessary to enable the Wi-Fi encryption selection within LuCI, any attempt to authenticate results in an immediate connection failure.
>
> **Error State:** The system consistently reports a kernel-level error (`23=IEEE8021X_FAILED`).
>
> **Isolation:** The failure state is specific to enterprise authentication. Testing connectivity against a standard WPA2-PSK network confirms that the Wi-Fi hardware and driver stack are operational, isolating the issue exclusively to the enterprise authentication workflow.

### 4.3 Root Cause Analysis

The Root Cause Analysis section states why the problem occurs. It contains three labeled parts:

- **Mechanism:** The subsystem or process involved.
- **The Problem:** The specific discrepancy, misconfiguration, or bug.
- **The Requirement:** What the system actually requires, and why the problem value fails to satisfy it.

Each labeled part is one to three sentences. The section must state the precise technical mismatch, not a general description.

Example:

> **Mechanism:** Authentication depends on the precise interpretation of the specified method.
>
> **The Problem:** The LuCI configuration generator contains a critical discrepancy. When the user selects the `EAP-MSCHAPv2` option, the generator inserts the literal string `EAP-MSCHAPV2` into the underlying `wpa_supplicant` configuration.
>
> **The Requirement:** The `wpa_supplicant` daemon requires the bare, canonical method identifier, `MSCHAPV2`. The inclusion of the `EAP-` prefix invalidates the configuration from the software's perspective, causing the authentication stage to fail silently.

### 4.4 Solution & Implementation Steps

The Solution section states how to fix the problem. It contains:

- **A one- to three-sentence description** of the approach, introduced by a bold label ("Implementation Protocol:" or a sentence stating what the solution involves).
- **Numbered steps.** Each step begins with a bold imperative label, followed by a description and, where applicable, a code block.

Each step must state what to do and, if non-obvious, why. Do not include exploratory commands or alternatives that were tried and abandoned; include only the commands required for the final solution.

Example:

> The solution requires overriding the flawed LuCI configuration generator by directly modifying the underlying system settings using the UCI command-line interface.
>
> **Implementation Protocol:**
>
> 1. **Identify STA Interface:** Use `uci show wireless | grep auth` to determine the correct index (`<index>`) for the client (STA) interface.
> 2. **Inject Correct Auth Value:** Apply the bare method name directly:
>    ```bash
>    uci set wireless.@wifi-iface[<index>].auth='MSCHAPV2'
>    ```
> 3. **Commit and Reload:** Finalize the changes and restart the network service:
>    ```bash
>    uci commit wireless
>    wifi reload
>    ```

### 4.5 Verification

The Verification section states how to confirm the fix worked. It is one to three sentences. If the verification requires a command, include it in a code block or inline.

Example:

> A successful connection is verified upon confirmation of a stable, authenticated connection to the enterprise network.

### 4.6 Caveats

Caveats describe limitations, side effects, or maintenance requirements. Use an Obsidian callout with the `warning` type:

```
> [!warning] **Caveat: Maintaining State**
> The web interface maintains a persistent state that enforces the invalid `EAP-MSCHAPV2` value. Any modification made within the LuCI GUI will revert the underlying configuration. To ensure continuous operation, the `uci` command sequence above must be reapplied whenever the Wi-Fi settings are modified or saved through the graphical interface.
```

Use a callout only when the caveat is critical (it causes failure if ignored). For minor notes, use a plain paragraph introduced by "**Note:**" in bold.

### 4.7 Prerequisites

List prior entries, tools, or access requirements as a bulleted list. Link prior entries using backlink syntax.

Example:

```
## Prerequisites

* [[Pt 1: Choosing the Router OS]] — OpenWrt image selected and the `.img.gz` file downloaded.
* [[Pt 2: Flashing and Booting]] — OpenWrt installed to the internal drive and booting.
```

### 4.8 Next Step

For series entries, end with a short sentence and a backlink to the next entry.

Example:

```
## Next Step

The system is now installed. The next step involves gaining initial access and setting the root password.

* [[Pt 3: Initial Access and Root Password]]
```

---

## 5. Sentence Construction Rules

### 5.1 Subject

The subject of a sentence is the system, the component, the configuration, or the procedure. Never the author.

- Wrong: "I set the value to `MSCHAPV2`."
- Right: "Set the value to `MSCHAPV2`." (imperative) or "The solution sets the value to `MSCHAPV2`." (declarative)

### 5.2 Voice

Use imperative mood for procedures and declarative mood for descriptions.

- Imperative: "Run `uci commit wireless` to apply the change."
- Declarative: "The daemon requires the bare method identifier."

Do not use passive voice unless the actor is unknown or irrelevant. "The value is written by the generator" is acceptable; "The value was set" (by whom?) is not.

### 5.3 Length

Keep sentences under 30 words where possible. If a sentence exceeds 30 words, split it.

### 5.4 Density

Every sentence must carry technical information. Remove sentences that only provide pacing, emphasis, or transition. If a sentence can be deleted without losing technical content, delete it.

### 5.5 Precision

Use the exact name of every technical entity: daemon, package, parameter, error code, file path, command. Do not paraphrase or abbreviate unless the abbreviation is defined in the same sentence.

- Wrong: "the wifi software"
- Right: "the `wpa_supplicant` daemon"

### 5.6 Cause and effect

When stating a behavior, state its cause. Use "because," "due to," or a semicolon.

- Weak: "The connection fails."
- Strong: "The connection fails because the daemon cannot parse the authentication method."

---

## 6. Technical Detail Requirements

### 6.1 Support claims with data

Every significant claim must be supported by a specific example, code snippet, parameter value, or error string. Do not make a claim that the reader cannot verify from the entry.

- Weak: "The configuration is invalid."
- Strong: "The configuration contains the string `EAP-MSCHAPV2`, which the daemon does not recognize."

### 6.2 Exact identifiers

When referring to a command, parameter, or error, use the exact string in inline code:

- `uci set wireless.@wifi-iface[1].auth='MSCHAPV2'`
- `23=IEEE8021X_FAILED`
- `wpa_supplicant`

### 6.3 Code blocks

Use fenced code blocks with a language tag for multi-line commands:

````
```bash
uci set wireless.@wifi-iface[1].auth='MSCHAPV2'
uci commit wireless
wifi reload
```
````

Use inline code (backticks) for single commands, parameters, filenames, and error strings.

### 6.4 Images

Preserve existing wiki-embed image syntax and italic captions:

```
![[filename.png]]
*Caption describing what the image shows.*
```

Do not add images. Do not remove images unless the image is no longer relevant to the rewritten content.

### 6.5 Frontmatter

Preserve the existing YAML frontmatter block. Update the `description` field to match the rewritten entry's scope if the original description is narrative or inaccurate. Keep `title` and `tags` unchanged.

---

## 7. Obsidian Formatting Rules

### 7.1 Internal links

Use backlink syntax for all internal wiki references:

```
[[Page Name]]
```

Do not use Markdown URLs for internal references:

```
[Page Name](path/to/page)  <-- wrong
```

Backlinks may include an alias:

```
[[Page Name|display text]]
```

### 7.2 Callouts

Use callouts only for critical warnings or mandatory protocol steps. The `warning` type is appropriate for caveats that cause failure if ignored.

```
> [!warning] **Caveat: Title**
> Body text.
```

Do not use callouts for emphasis, decoration, or non-critical notes. Do not use callouts for information that fits in a plain paragraph.

### 7.3 Bold lead-in labels

Use bold text followed by a colon to introduce a labeled part of a section:

```
**Error State:** The system reports ...
**Isolation:** Testing against ...
**Mechanism:** Authentication depends on ...
```

This is the primary structural device of the blended style. Use it whenever a section contains more than one distinct fact or mechanism.

### 7.4 Tables

Use a table only when comparing three or more items across two or more attributes. Do not use a table for two-item comparisons; use prose.

### 7.5 Advanced syntax

Use Obsidian-flavored Markdown (callouts, embeds, comments) only when it improves comprehension. Do not use advanced syntax for stylistic effect. If plain Markdown achieves the same clarity, use plain Markdown.

---

## 8. Transformation Patterns

The following table shows common transformations from a narrative or first-person style to the blended style.

| Original (do not write this) | Rewritten (write this) |
|---|---|
| "I found that connecting to the network required installing `wpa_supplicant`." | "The `wpa_supplicant` package is required for LuCI to present Wi-Fi encryption options." |
| "I spent six hours working through it with Claude." | (Delete. Not technically relevant.) |
| "LuCI's config generator has a bug: when I select EAP-MSCHAPv2, it writes the literal string..." | "The LuCI configuration generator inserts the literal string `EAP-MSCHAPV2` into the underlying `wpa_supplicant` configuration when the `EAP-MSCHAPv2` option is selected." |
| "I set the underlying UCI value directly, bypassing the buggy generator." | "Set the underlying UCI value directly to bypass the configuration generator." |
| "I have to reapply this every time I change Wi-Fi settings from LuCI." | "The `uci` command sequence must be reapplied whenever Wi-Fi settings are modified or saved through the LuCI interface." |
| "The stick didn't appear at all in the boot menu because the laptop's BIOS screen was actually a UEFI menu." | "The USB stick did not appear in the boot menu because the host's boot interface was UEFI, not BIOS, and the image was the non-EFI variant." |
| "Booting the EFI image got me as far as a shell prompt." | "The EFI image boots to a shell prompt." |
| "I abandoned Ventoy as the boot medium for this image." | "Ventoy was discarded as a viable boot medium for this specific image variant." |

---

## 9. Worked Example

The following is a complete entry written in the target style. Use it as a reference for structure, tone, and formatting.

````markdown
# Enterprise Wi-Fi (WPA2-EAP) Authentication

---
title: Enterprise Wi-Fi (WPA2-EAP) authentication
description: A technical guide to successfully connecting an OpenWrt device to WPA2-Enterprise networks.
tags:
  - Homelab
  - OpenWrt
---

## Overview

This guide details the process of successfully connecting an OpenWrt device to a WPA2-EAP (PEAP / MSCHAPv2) enterprise network, addressing a known bug within the system's configuration generator.

## Observed Issue: Authentication Failure

The system fails to establish a connection when targeting a WPA2-EAP (PEAP/MSCHAPv2) enterprise network. While the installation of the `wpa_supplicant` package is necessary to enable the Wi-Fi encryption selection within LuCI, any attempt to authenticate results in an immediate connection failure.

**Error State:** The system consistently reports a kernel-level error (`23=IEEE8021X_FAILED`).

**Isolation:** The failure state is specific to enterprise authentication. Testing connectivity against a standard WPA2-PSK network confirms that the Wi-Fi hardware and driver stack are operational, isolating the issue exclusively to the enterprise authentication workflow.

## Root Cause Analysis: Configuration Discrepancy

**Mechanism:** Authentication depends on the precise interpretation of the specified method.

**The Problem:** The LuCI configuration generator contains a critical discrepancy. When the user selects the `EAP-MSCHAPv2` option, the generator inserts the literal string `EAP-MSCHAPV2` into the underlying `wpa_supplicant` configuration.

**The Requirement:** The `wpa_supplicant` daemon requires the bare, canonical method identifier, `MSCHAPV2`. The inclusion of the `EAP-` prefix invalidates the configuration from the software's perspective, causing the authentication stage to fail silently.

## Solution & Implementation Steps: Forcing the Correct State

The solution requires overriding the flawed LuCI configuration generator by directly modifying the underlying system settings using the UCI command-line interface.

**Implementation Protocol:**

1. **Identify STA Interface:** Use `uci show wireless | grep auth` to determine the correct index (`<index>`) for the client (STA) interface.
2. **Inject Correct Auth Value:** Apply the bare method name directly:
    ```bash
    uci set wireless.@wifi-iface[<index>].auth='MSCHAPV2'
    ```
3. **Commit and Reload:** Finalize the changes and restart the network service:
    ```bash
    uci commit wireless
    wifi reload
    ```

> [!warning] **Caveat: Maintaining State**
> The web interface maintains a persistent state that enforces the invalid `EAP-MSCHAPV2` value. Any modification made within the LuCI GUI will revert the underlying configuration. To ensure continuous operation, the `uci` command sequence above must be reapplied whenever the Wi-Fi settings are modified or saved through the graphical interface.

## Verification

A successful connection is verified upon confirmation of a stable, authenticated connection to the enterprise network.
````

---

## 10. Precedence Rules

This section defines how to resolve conflicts between rules.

1. **The blended style takes absolute priority.** The style described in Sections 1, 4, 5, and 6 (Arch Wiki precision blended with Minecraft Wiki explanatory structure) is the primary directive. If any rule in this guide conflicts with the blended style, the blended style wins.
2. **The absolute prohibitions take priority over section requirements.** If a section requirement would require an emoji or an em dash, do not satisfy the requirement; find another way.
3. **The Obsidian formatting rules are subordinate to the style.** Use Obsidian features only when they improve comprehension. Never use a feature because the rules mention it.
4. **The worked example is illustrative, not exhaustive.** Follow the patterns in the worked example, but do not copy its content. Apply the patterns to the entry being rewritten.

---

## 11. Final Checklist

Before returning a rewritten entry, verify every item:

1. [ ] No emojis anywhere in the document.
2. [ ] No em dashes or en dashes used as punctuation.
3. [ ] No first-person pronouns or verbs.
4. [ ] No narrative of discovery (no "I found," "I spent," "I tried").
5. [ ] No metaphors, similes, or clichés.
6. [ ] No "not just X, but also Y" constructions.
7. [ ] No concluding or transitional filler phrases.
8. [ ] No source footer.
9. [ ] Prerequisites section is placed correctly (before Overview for series steps, after Overview for tools).
10. [ ] Overview is one to three sentences and states scope only.
11. [ ] Observed Issue contains the failure description, error state, and isolation.
12. [ ] Root Cause Analysis contains Mechanism, The Problem, and The Requirement.
13. [ ] Solution contains numbered imperative steps with code blocks.
14. [ ] Verification states how to confirm success.
15. [ ] Caveats use a callout only if critical.
16. [ ] All internal links use `[[backlink]]` syntax.
17. [ ] All technical identifiers are in inline code.
18. [ ] Every significant claim is supported by a specific example, parameter, or error string.
19. [ ] No sentence can be deleted without losing technical content.
20. [ ] The entry reads as a technical specification, not a personal blog post.
