---
title: LightRAG Parser Routing and the ! Option
description: LightRAG parser routing configuration, the ! option for skipping extraction, and supported file types.
tags:
  - LightRAG
  - homelang
  - fedora
  - rag
  - routing
---

## Overview

This page documents the LightRAG parser routing configuration on the Local-AI-VM Fedora platform. LightRAG uses the `LIGHTRAG_PARSER` environment variable to route different file types to different processing engines with specific options.

---

## Routing Syntax

`LIGHTRAG_PARSER` is a comma- or semicolon-separated list of rules. Each rule is `<suffix-list>:<engine><options>`:

```
png:docling-iP!,jpg:docling-iP!,jpeg:docling-iP!,*:native-iP,*:docling-iP,*:legacy-R
```

Rules are evaluated left to right; the first match wins. Extension-specific rules come before `*` wildcards.

### Option Letters

| Letter | Meaning |
|---|---|
| `i` | Analyze images with the VLM |
| `t` | Analyze tables with the extraction LLM |
| `e` | Analyze equations |
| `P` | Paragraph-semantic chunking |
| `R` | Recursive character chunking (legacy default) |
| `F` | Fixed-token chunking |
| `V` | Semantic vector chunking |
| `!` | **Skip entity/relation extraction and graph writing**; chunks are still vector-indexed |

Per-rule parameters can be attached: `P(chunk_ts=800,chunk_ol=80)`; a boolean parameter as a bare flag: `P(drop_rf)`.

---

## What ! Does and Does Not Do

The `!` option applies at the extraction stage. It:

- **Skips** entity extraction, relation extraction, entity merge, relation merge, and graph writes for the affected document.
- **Still runs** parsing (including Docling), VLM analysis (if `i` is set), chunking, and embedding.
- **Still produces** two chunks per image: `chunk-000` (the block text — the `<drawing .../>` tag or the heading) and `mm-drawing-000` (the VLM description paragraph).

### Retrieval Modes

| Mode | Uses graph | Uses chunks |
|---|---|---|
| naive | no | yes |
| local | yes | no |
| global | yes | no |
| hybrid | yes | no |
| mix | yes | yes |

The WebUI uses `mix` by default. A `!`-routed image is retrievable through the vector half (`naive` / `mix`). Graph-only modes (`local` / `global` / `hybrid`) will not find it.

**Does NOT prevent the zero-blocks error** — that error happens at the parse stage, before `!` is ever consulted.

---

## Per-File Hints

For files where the routing rule isn't enough, embed an override in the filename:

```
name.[-iP!].md
```

Options in `[...]` **replace** the rule's options, they do not add. So `[-iP!]` runs VLM analysis (`i`), paragraph chunking (`P`), and skips extraction (`!`). If `i` is left out, the VLM will not run — a common gotcha.

**For wrapped images** (`wrap_images.py` era), the Markdown file needs a per-file hint because the wildcard `*` rule routes `.md` to `native-iP` (extraction on). Adding `[-iP!]` to the wrapped filename suppresses extraction for that one file.

---

## Current Routing Rule

Deployed `.env`:

```
LIGHTRAG_PARSER=png:docling-iP!,jpg:docling-iP!,jpeg:docling-iP!,*:native-iP,*:docling-iP,*:legacy-R
```

**What it does:**

| Suffix | Engine | Options | Effect |
|---|---|---|---|
| `.png`, `.jpg`, `.jpeg` | Docling | `iP!` | VLM description + paragraph chunking, no KG extraction |
| `*` (first fallback) | `native` | `iP` | Handles `.md`, `.docx`, `.textpack` — VLM + paragraph chunking, extraction on |
| `*` (second fallback) | Docling | `iP` | Handles `.pdf`, `.docx`, `.pptx`, `.xlsx`, etc. |
| `*` (last) | `legacy` | `R` | Plain text files, prose, etc. |

### Gaps in the ! Rule

- `.tiff`, `.webp`, `.bmp` are Docling-supported but have no `!` rule, so they fall through to `*:docling-iP` with extraction **on**. This is a hallucination risk. If the user re-enables image ingestion, they should extend the `!` rule to include `tiff:docling-iP!,webp:docling-iP!,bmp:docling-iP!`.
- PDF-embedded images also go through `*:docling-iP` without `!`, so they still get extracted into the KG. This can produce minor noise (e.g., "Red Brick Wall" or "Blue Line" from literal VLM descriptions).

---

## Supported File Types (Live)

From `/documents/supported_file_types`:

```json
{
  "supported_extensions": [".bat",".bmp",".c",".conf",".cpp",".css",".csv",".docx",".epub",".go",".h",".hpp",".htm",".html",".ini",".java",".jpeg",".jpg",".js",".json",".less",".log",".md",".mdx",".odt",".pdf",".php",".png",".pptx",".properties",".py",".rb",".rtf",".scss",".sh",".sql",".swift",".tex",".textpack",".tiff",".ts",".txt",".webp",".xhtml",".xlsx",".xml",".yaml",".yml"],
  "engines": {
    "docling": [".bmp",".docx",".html",".jpeg",".jpg",".md",".pdf",".png",".pptx",".tiff",".webp",".xhtml",".xlsx"],
    "legacy":  [".bat",".c",".conf",".cpp",".css",".csv",".docx",".epub",".go",".h",".hpp",".htm",".html",".ini",".java",".js",".json",".less",".log",".md",".mdx",".odt",".pdf",".php",".pptx",".properties",".py",".rb",".rtf",".scss",".sh",".sql",".swift",".tex",".ts",".txt",".xlsx",".xml",".yaml",".yml"],
    "native":  [".docx",".md",".textpack"]
  }
}
```

Note: `.png`, `.jpg`, `.jpeg` are in the Docling engine list only because the `.env` routes them there. Without a Docling service, image extensions would not appear.
