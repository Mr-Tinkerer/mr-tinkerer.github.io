# Quartz Wiki Post-Write Tweak Overview

This document contains all correction guidelines and new rules developed for improving your Quartz Obsidian wiki's discoverability, searchability, and overall structure.

---

## 🎯 Primary Goals

1. **Improve searchability** - Every page must be uniquely discoverable via search
2. **Fix structural issues** - Add missing Index.md files and standardize naming
3. **Enhance navigation** - Ensure all content is properly linked and organized
4. **Standardize conventions** - Apply consistent naming, headings, and formatting

---

## 🔴 Critical Issues to Fix (High Priority)

### 1. Missing Platform VM Index Files

**Problem:** Productivity-VM, Local-AI-VM, and Personal-VM have content but NO `Index.md` files.

**Rule:** Every top-level domain folder MUST have exactly one `Index.md` (per §4).

**Action Required:**
- Create `Platform/Productivity-VM/Index.md`
- Create `Platform/Local-AI-VM/Index.md`
- Create `Platform/Personal-VM/Index.md`

**Content Requirements for VM Index.md:**
- Brief description of the VM's role/purpose
- List all OS and service buckets it contains
- Links to each bucket's documentation
- Cross-references between services
- Performance considerations
- Future expansion notes

---

### 2. Empty Top-Level Bucket Directories

**Problem:** Hardware/, Software/, Logical/, and Platform/ are currently EMPTY directories.

**Rule:** Top-level buckets must have exactly one `Index.md` that summarizes all content.

**Action Required:**
- Create `Homelab/Hardware/Index.md` - Summary of all hardware entries
- Create `Homelab/Software/Index.md` - Summary of all software entries
- Create `Homelab/Logical/Index.md` - Summary of all logical topics
- Create `Homelab/Platform/Index.md` - Summary of all Platform VMs

**Content Requirements:**
- Introduction to the domain
- One `#` heading per bucket type
- Bullet list of all buckets with links
- Cross-links between related buckets
- **NO nested subheadings** (keep it flat)

---

### 3. Generic Page Titles

**Problem:** Many pages have vague titles like "Overview", "Setup", "Installation" without specific context.

**Rule:** Both frontmatter `title:` and filename must identify what the entry is *about*, not just what kind of entry it is (per §16a).

**Examples of Bad → Good:**
- `Overview.md` → `Dell-Optiplex-5060 Overview.md`
- `Installation.md` → `Nextcloud Installation.md`
- `Setup.md` → `OpenWrt Setup.md`
- `Configuration.md` → `Proxmox Configuration.md`

**Decision Rule:** "If I searched for this title with no folder context, would I know which entry it is?"

---

### 4. Pages with No Headings

**Problem:** Several pages have 0 headings despite being 500+ characters.

**Rule:** Content pages should have minimum 2 headings for proper structure.

**Action Required:**
- Add proper section headings to all pages with 0-1 headings
- Use `---` to separate major sections
- Structure: Introduction → Main Content → Troubleshooting/Notes

---

## 📊 Structural Standards (Medium Priority)

### 5. Platform VM Structure Clarification

**Current structure (CORRECT):**
```
Platform/
├── Productivity-VM/
│   ├── Debian/            ← OS bucket
│   ├── Nextcloud/         ← Service bucket
│   ├── Stirling-pdf/      ← Service bucket
│   └── Convertx/          ← Service bucket
└── Local-AI-VM/
    ├── Fedora/            ← OS bucket
    ├── Llama.cpp/         ← Service bucket
    ├── Open-webui/        ← Service bucket
    └── Searxng/          ← Service bucket
```

**This is GOOD and should be maintained.**

**Enhancement:** Add subdirectories for growing services:
```
Nextcloud/
├── Installation/          ← Sequential steps
│   ├── Pt 1, Prerequisites.md
│   ├── Pt 2, Initial-setup.md
│   └── Pt 3, Configuration.md
├── Usage/                 ← Reference topics
│   ├── File-sharing.md
│   └── Collaboration.md
├── Troubleshooting/       ← Gotchas
│   └── Gotchas.md
└── Overview.md
```

---

### 6. Naming Convention Standardization

**Current Issue:** Mixed case for Linux distros (Debian/, Fedora/, Opensuse/)

**Rule:** Pick ONE convention and stick to it.

**Option A - Lowercase (recommended for Linux distros):**
- `debian/`, `fedora/`, `opensuse/`, `ubuntu/`
- Matches: `nginx/`, `tailscale/`, `openwrt/`

**Option B - Capitalized:**
- `Debian/`, `Fedora/`, `Opensuse/`
- Matches: `Nextcloud/`, `OpenWrt/`, `Proxmox/`

**Recommendation:** Use **Option A (lowercase)** for Linux distros to maintain consistency with other services.

---

### 7. Directory Depth Limits

**Current Maximum:** 3 levels (Platform/Productivity-VM/Nextcloud/)

**Rule:** Maximum depth = 4 levels

**Examples of acceptable depth:**
- `Platform/Productivity-VM/Nextcloud/Installation/` (4 levels) ✓
- `Hardware/Dell-Optiplex-5060/Build-and-repairs/` (3 levels) ✓

**Avoid:**
- Excessive subdirectories for simple services
- Deep nesting makes navigation harder

---

## ✍️ Content Quality Standards (Low Priority)

### 8. Frontmatter Completeness

**Rule:** Every content page must have complete frontmatter:
```yaml
---
title: [Specific Page Title]
description: [Brief description - required]
tags:
  - [relevant tags]
---
```

**Current Issue:** Some pages have incomplete descriptions.

---

### 9. Cumulative Prerequisites in Sequences

**Rule:** Every page in a sequential workflow must link ALL prior steps.

**Example:**
```markdown
# Pt 3, Configuration.md

## Prerequisites
- [[Homelab/Software/OpenWrt/Installation/Pt 1, Prerequisites|Pt 1]]
- [[Homelab/Software/OpenWrt/Installation/Pt 2, Fixing-content-rendering|Pt 2]]
```

---

### 10. Heading Structure

**Rule:** Use `---` to separate major sections.

**Bad:**
```markdown
# Title

Some text...

More text...

# Another section
```

**Good:**
```markdown
# Title

Some text...

---

## Another section

More text...

---

## Troubleshooting

Known issues...
```

---

## 🔍 Search-Specific Naming Rules

### 11. Filename Requirements (per §16a)

**Bad filenames:**
- `Glossary.md` → `Nextcloud Glossary.md` or `Nextcloud-glossary.md`
- `Gotchas.md` → `OpenWrt Gotchas.md` or `Opensuse-gotchas.md`
- `Overview.md` → `Dell-Optiplex-5060 Overview.md`

**Decision Rule:** "If I searched for this title in a search bar with no folder context, would I know which entry it is?"

---

### 12. Index.md Title Requirements

**Rule:** Index.md files must have specific titles.

**Good examples:**
- `Homelab Index.md`
- `Hardware Overview.md`
- `Productivity-VM Overview.md`

**Bad examples:**
- `Index.md` (too generic)
- `Overview.md` (too generic)

---

## 📋 Pre-Creation Checklist (Updated)

Before creating any new bucket or page:

- [ ] Is it about one specific item, OS, service, or tool?
- [ ] Could an existing bucket own this page?
- [ ] Would merging it into an existing bucket improve discoverability?
- [ ] Am I accidentally recreating the structure of a legacy blog post?
- [ ] Is this actually a sequence? If yes, does it have its own subfolder?
- [ ] Is the content duplicated elsewhere?
- [ ] Does the bucket need a glossary entry rather than an inline definition?
- [ ] **NEW:** For top-level buckets: Does this need its own `Index.md`?
- [ ] **NEW:** For Platform VMs: Will this VM need its own `Index.md`?
- [ ] **NEW:** Is the filename specific enough to be search-discoverable?
- [ ] **NEW:** Will this page have at least 2 headings?
- [ ] **NEW:** For sequences: Are cumulative prerequisites set up correctly?

---

## 🧪 Verification Checklist (Before Finalizing)

Before finalizing any changes:

- [ ] Every top-level domain has exactly one `Index.md`
- [ ] No normal bucket has an `Index.md`
- [ ] Every bucket has a `Glossary.md`
- [ ] Every sequential workflow has its own folder
- [ ] Every sequential page has cumulative prerequisites
- [ ] No duplicate procedures exist
- [ ] **NEW:** All Platform VM directories have `Index.md` files
- [ ] **NEW:** All top-level buckets (Hardware/Software/Logical/Platform) have `Index.md` files
- [ ] **NEW:** All pages have search-specific titles (no generic names)
- [ ] **NEW:** All content pages have at least 2 headings
- [ ] **NEW:** All Index.md files have specific titles
- [ ] **NEW:** Naming conventions are consistent (choose lowercase or capitalized for distros)
- [ ] **NEW:** No orphan directories (empty folders)
- [ ] Every kept image lives in its domain's shared image folder
- [ ] Every wikilink to a moved or renamed page still resolves
- [ ] Any unresolved ambiguity has a targeted question in `Migration Q&A.json`

---

## 📝 New Rules to Add to Main Prompts

### New §21. Directory Structure Standards

```markdown
## 21. Directory Structure Standards

### Top-level buckets (Hardware/Software/Logical/Platform)
- Each top-level bucket MUST have exactly one `Index.md` file
- The `Index.md` must summarize all content buckets within that domain
- No normal bucket gets its own `Index.md` (only top-level domains)
- These Index files serve as navigation hubs, not content pages

### Platform VM structure
- Each VM directory (e.g., Productivity-VM/) MUST have its own `Index.md`
- VM Index must list all OS and service buckets it contains
- VM Index must link to each bucket's documentation
- VM Index must explain the VM's role in the home lab

### OS vs Service separation
- Each VM runs exactly one OS, which gets its own bucket (e.g., Debian/)
- Services running on that VM get their own buckets (e.g., Nextcloud/)
- OS buckets should be flat (no subdirectories) unless the OS has major subsystems
- Service buckets follow flat reference or sequence rules
```

### New §22. Index.md Requirements

```markdown
## 22. Index.md Requirements

### Top-level domain Index.md
- Introduction to the domain
- One `#` heading per bucket type
- Bullet list of all buckets with links to their `Glossary.md` and `Overview.md`
- Cross-links between related buckets
- No nested subheadings (keep it flat)

### Platform VM Index.md
- VM role and purpose
- Guest OS overview (link to OS bucket)
- List of all services running on this VM
- Service dependencies and relationships
- Links to each service bucket's documentation
- Performance considerations
- Future expansion notes

### Verification checklist before finalizing:
- [ ] Every top-level domain has exactly one `Index.md`
- [ ] Every Platform VM has its own `Index.md`
- [ ] No normal bucket has an `Index.md`
- [ ] All Index.md files link to their child buckets
- [ ] Index.md files are summaries, not full documentation
```

### New §23. Directory Depth Limits

```markdown
## 23. Directory Depth Limits

- Maximum directory depth: 4 levels (e.g., Platform/Productivity-VM/Nextcloud/Installation/)
- Avoid creating subdirectories for simple services
- If a bucket has >3 files, consider organizing into subdirectories
- Keep the structure shallow for better navigation
- Deep nesting makes the graph hard to understand
```

---

## 🚀 Action Plan

### Phase 1: Critical Fixes (Do Immediately)
1. Create all missing Platform VM Index.md files
2. Create all missing top-level bucket Index.md files
3. Rename all generic page titles to be search-specific
4. Add headings to pages with 0-1 headings

### Phase 2: Structural Cleanup
1. Standardize OS naming convention (lowercase recommended)
2. Remove orphan directories
3. Consider adding subdirectories for growing services

### Phase 3: Content Quality
1. Review all pages for complete frontmatter
2. Ensure cumulative prerequisites in sequences
3. Apply `---` separators between major sections

### Phase 4: Verification
1. Run full verification checklist
2. Test search functionality
3. Verify all wikilinks resolve

---

## 📚 Reference Files

- `quartz-docs-rules.md` - Main migration rules
- `quartz-docs-rules-new-entries.md` - Adding new content rules
- `homelab-platform-buckets-rule.md` - Platform VM bucket convention
- `post-write-tweak-overview.md` - **This document**

---
