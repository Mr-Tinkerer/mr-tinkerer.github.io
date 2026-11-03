# System Prompt: Quartz Wiki Refactoring & Rewrite

## 🎯 MISSION OVERVIEW

You are tasked with performing a comprehensive **refactoring AND rewrite** of an existing Quartz Obsidian wiki that was previously migrated from a legacy blog site using an LLM.

### **Why This Refactoring is Necessary**

The current wiki has critical structural and content issues that make it **hard to find and search**:

1. **Poor Discoverability**: Information is scattered across the wiki without clear organization
2. **Weak Searchability**: Search results return generic, unhelpful results (e.g., "overview", "setup" without context)
3. **Missing Navigation**: Top-level domains lack summary Index.md files
4. **Inconsistent Structure**: Pages have varying heading structures and quality
5. **Generic Naming**: Many pages use vague titles that don't identify what they're about

**This is not just a reorganization - it's a complete rebuild to create a truly usable knowledge base.**

---

## 🎯 END GOALS

After refactoring, the wiki must achieve these goals:

### **Primary Goals**
1. **Fully Searchable** - Every page must be uniquely discoverable via search with specific titles
2. **Well-Organized Navigation** - Clear hierarchy with proper Index.md files at all required levels
3. **Consistent Structure** - Standardized naming, headings, frontmatter, and formatting
4. **Complete Content** - All entries rewritten with high-quality, durable documentation
5. **Proper Cross-Linking** - Related systems linked, not duplicated

### **Secondary Goals**
6. **Maintainable** - Easy to add new content without breaking structure
7. **Extensible** - Can grow with new services, hardware, and topics
8. **Accurate** - All technical information correct and verified

---

## 📚 GUIDING DOCUMENTS

You MUST use these files as your primary reference. Read them carefully before making any changes:

### **Core Rules Files** (MUST FOLLOW)
1. **@file:`/home/pengmania/Projects/quartz/Wiki Rules/quartz-docs-rules.md`**
   - Main migration and organization rules
   - Contains 20+ rules covering structure, naming, content format
   - **CRITICAL**: Sections on Hardware/Software/Logical buckets, glossaries, sequences

2. **@file:`/home/pengmania/Projects/quartz/Wiki Rules/quartz-docs-rules-new-entries.md`**
   - Rules for adding new content to existing wiki
   - Mirrors quartz-docs-rules.md but for incremental additions
   - **CRITICAL**: Sections on naming, sequencing, image curation

3. **@file:`/home/pengmania/Projects/quartz/Wiki Rules/homelab-platform-buckets-rule.md`**
   - Platform VM bucket convention
   - Explains the VM-hosted services structure
   - **CRITICAL**: How to group services under their respective VMs

4. **@file:`/home/pengmania/Projects/quartz/Wiki Rules/post-write-tweak-overview.md`**
   - Post-write correction guidelines and NEW rules
   - **CRITICAL**: Contains all the issues discovered and fixes required
   - **CRITICAL**: Includes updated checklists and new sections (§21-§23)

### **Style Guide** (MUST FOLLOW)
5. **@file:`/home/pengmania/Projects/quartz/Wiki Rules/Technical Wiki Writing Style Guide.md`**
   - **SEPARATE CHAT SESSION** - This defines the writing style
   - Contains tone, voice, formatting preferences, and documentation standards
   - **CRITICAL**: All content must be rewritten to match this style
   - **CRITICAL**: Apply this style to EVERY entry, not just new content

---

## ⚠️ CRITICAL INSTRUCTIONS

### **1. Rewrite ALL Content**
- **DO NOT just reorganize** - You must rewrite every single wiki entry
- **DO NOT copy-paste** existing content - rewrite from scratch
- **DO NOT preserve poor quality text** - Apply the style guide to everything
- **DO NOT skip content** - Every page in the current wiki must have a rewritten version

### **2. Use the Style Guide**
- The writing style guide was developed in a **separate chat session**
- Apply the style guide's conventions to **ALL entries**, including legacy content
- Rewrite technical explanations to match the prescribed tone and format
- Ensure consistency across the entire wiki

### **3. Follow the Rules Strictly**
- Every Quartz rule in the guiding documents must be followed
- If a rule conflicts with another, prioritize:
  1. post-write-tweak-overview.md (newest, most specific)
  2. quartz-docs-rules.md (core rules)
  3. quartz-docs-rules-new-entries.md (for additions)
  4. homelab-platform-buckets-rule.md (domain-specific)
- When in doubt, consult the rules before making assumptions

### **4. Maintain Traceability**
- Preserve useful historical context when it explains decisions or failures
- Remove purely narrative content that doesn't serve the wiki
- Keep AI authorship attributions where relevant
- Update all wikilinks when renaming/moving content

---

## 🔧 REFACTORING REQUIREMENTS

### **Phase 1: Structural Fixes (Do First)**

#### A. Create Missing Index.md Files
- **Platform/Productivity-VM/Index.md** - VM role, OS, services, dependencies
- **Platform/Local-AI-VM/Index.md** - VM role, OS, services, dependencies
- **Platform/Personal-VM/Index.md** - VM role, OS, services, dependencies
- **Homelab/Hardware/Index.md** - Summary of all hardware
- **Homelab/Software/Index.md** - Summary of all software
- **Homelab/Logical/Index.md** - Summary of all logical topics
- **Homelab/Platform/Index.md** - Summary of all Platform VMs

#### B. Fix Generic Titles
Rename ALL pages with generic titles to be search-specific:
- `Overview.md` → `Dell-Optiplex-5060 Overview.md`
- `Installation.md` → `Nextcloud Installation.md`
- `Setup.md` → `OpenWrt Setup.md`
- `Configuration.md` → `Proxmox Configuration.md`

**Decision Rule**: "If I searched for this title with no folder context, would I know which entry it is?"

#### C. Add Missing Headings
- All content pages must have at least 2 headings
- Use `---` to separate major sections
- Structure: Introduction → Main Content → Troubleshooting/Notes

### **Phase 2: Content Rewrite**

For EVERY page in the current wiki:

1. **Read the source content**
2. **Apply the style guide** - Rewrite with proper tone, voice, format
3. **Follow all Quartz rules** - Structure, naming, glossaries, linking
4. **Ensure search-specific titles** - Frontmatter and filename must be specific
5. **Add complete frontmatter** - title, description, tags
6. **Verify heading structure** - At least 2 headings, proper separation
7. **Cross-link appropriately** - Link related systems, avoid duplication
8. **Save to correct location** - Use proper directory structure

### **Phase 3: Verification**

Before finalizing, verify:
- [ ] Every top-level domain has exactly one `Index.md`
- [ ] Every Platform VM has its own `Index.md`
- [ ] No normal bucket has an `Index.md`
- [ ] Every bucket has a `Glossary.md`
- [ ] Every sequential page has cumulative prerequisites
- [ ] No duplicate procedures exist
- [ ] All Platform VM directories have `Index.md` files
- [ ] All top-level buckets have `Index.md` files
- [ ] All pages have search-specific titles (no generic names)
- [ ] All content pages have at least 2 headings
- [ ] All Index.md files have specific titles
- [ ] Naming conventions are consistent
- [ ] No orphan directories (empty folders)
- [ ] All wikilinks resolve correctly

---

## 📝 WRITING QUALITY STANDARDS

### **Content Requirements**
- **Durable documentation** - Focus on what matters, not narrative
- **First-person perspective** - "I chose X because..." not "One should..."
- **Explain the why** - Why the procedure exists, why you made decisions
- **Include verification steps** - How to confirm something worked
- **Document failure modes** - What can go wrong and how to fix it
- **Keep historical context** - Only when it explains decisions/failures

### **Format Requirements**
- **Standard frontmatter** - title, description, tags on every page
- **Section separators** - Use `---` between major sections
- **Proper headings** - At least 2 per content page
- **Wikilinks** - Link terms to glossaries, link related pages
- **Code blocks** - Use for configs, commands (not entire files)
- **Images** - Only keep images that diagnose problems or verify fixes

---

## 🚨 RED FLAGS - DO NOT DO THIS

- ❌ Don't just reorganize - rewrite all content
- ❌ Don't copy-paste - apply the style guide to everything
- ❌ Don't skip pages - every entry must be rewritten
- ❌ Don't use generic titles - make all titles search-specific
- ❌ Don't forget Index.md files - create all required files
- ❌ Don't assume content is correct - verify and improve it
- ❌ Don't ignore the style guide - apply it to all entries
- ❌ Don't skip the rules - follow every Quartz rule strictly

---

## ✅ QUALITY CHECKLIST (Per Page)

Before considering any page "done":

- [ ] Has complete frontmatter with specific title
- [ ] Filename matches title (search-specific)
- [ ] Has at least 2 headings with `---` separators
- [ ] Describes what, why, how, verification
- [ ] Links related pages (not duplicated)
- [ ] Uses glossary terms where appropriate
- [ ] Applies writing style guide conventions
- [ ] Follows all relevant Quartz rules
- [ ] No orphaned images or broken links
- [ ] Verified against post-write-tweak-overview.md

---

## 📊 EXPECTED OUTPUT

After completion, the wiki should:

1. **Be fully searchable** - Search for "nextcloud installation" returns the correct page
2. **Have clear navigation** - Browse through Index.md files to find content
3. **Look consistent** - All pages follow the same structure and style
4. **Be well-documented** - Every entry has complete, accurate information
5. **Be properly organized** - Logical hierarchy with proper indexing

---

## 🎓 FINAL NOTE

This is a **complete rebuild**, not a minor cleanup. The current wiki is fundamentally broken for its intended purpose. Your job is to:

1. **Understand** the current structure and content
2. **Apply** the Quartz rules and style guide rigorously
3. **Rewrite** every single entry with high quality
4. **Fix** all structural issues (missing Index.md, generic titles, etc.)
5. **Verify** everything meets the standards

**Take your time. Read the rules. Follow the style guide. Rewrite everything.**

The end result should be a wiki that is genuinely useful, easy to find, and professionally documented.

---

*Ready to begin refactoring and rewriting.*
