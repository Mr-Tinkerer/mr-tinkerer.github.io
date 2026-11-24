---
title: Docling Document Parsing
description: Docling document parsing configuration, the 90% full-bleed filter fix, and OCR settings on the Local-AI-VM.
tags:
  - Docling
  - LightRAG
  - homelab
  - fedora
  - parsing
---

## Overview

This page documents the Docling document parsing configuration on the Local-AI-VM Fedora platform. Docling is a document parsing library that converts PDFs and other document formats into structured data (blocks, drawings, tables). It uses layout detection to identify different content types and can route documents to different processing engines.

---

## The 90% Full-Bleed Filter (Root Cause)

### Symptoms and Isolating Experiment

- `Tux with Gun.png` (329×330, RGBA, full-bleed cartoon) failed with Docling IR builder produced zero blocks.
- Tests that did not fix it: 4× upscale, square canvas, remove alpha channel, all combinations.
- Tests that did fix it: add a white margin around the image; draw text on it (but this only produced OCR text — the picture was still dropped).
- Adding text and getting a text block "TUX" was a silent partial success — LightRAG status said success but `drawing_file: false` and `asset_dir: false`. The VLM never ran.

The isolating diagnostic was a monkeypatch inside the container that printed every layout cluster before the post-processor filter ran:

```
PAGE 78.96 79.20
regular_clusters text    conf=0.743 area/page=0.965
special_clusters picture conf=0.924 area/page=0.999
RESULT pictures: 0 texts: 1
```

Heron detected the picture with 0.924 confidence but its bounding box covered 99.9% of the page. The `layout_postprocessor.py` rule drops any `PICTURE` cluster with `area/page > 0.90`. For a standalone full-bleed image — which Docling treats as a one-page document — the picture is the page.

The leftover text cluster (conf 0.743, 96.5%) is an OCR hallucination that produced the UI's stray "A".

### Fix Options

1. **Patch the source** (chosen). Raise `0.90` to `2.0` in `layout_postprocessor.py`. Full mechanism and guards in the Containerfile.
2. **Pad the image** onto a white canvas before ingestion. Works but requires per-file user action.
3. **Bypass Docling via Markdown wrapper** (`wrap_images.py`) — embed the image as a base64 data URL in a `.md` file; `native` engine sends the whole image to the VLM with no layout detection. Retired when the patch was proven.

### Trade-off

The same filter also prevents a scanned PDF page from being treated as a single giant figure. With it disabled, a scan may parse as a picture rather than as OCR'd text. Test scanned PDFs after any docling-serve upgrade.

### Guards

The two `grep -q` steps fail the build if upstream renames the file, rewrites the comparison, or the `sed` fails to substitute. `python -m py_compile` guards against a syntactically broken patch.

---

## Containerfile (Docling Patch Source)

```dockerfile
FROM ghcr.io/docling-project/docling-serve-cpu:latest
USER 0
RUN f=/opt/app-root/lib64/python3.12/site-packages/docling/utils/layout_postprocessor.py && \
    grep -q 'cluster.bbox.area() / page_area > 0.90' "$f" && \
    sed -i 's|cluster.bbox.area() / page_area > 0.90|cluster.bbox.area() / page_area > 2.0|' "$f" && \
    grep -q 'page_area > 2.0' "$f" && \
    python -m py_compile "$f"
USER 1001
```

**Note on `:latest`:** User reports the live image is digest-pinned. The uploaded reference file still shows `:latest`; the live `Containerfile` should be updated to `FROM ghcr.io/docling-project/docling-serve-cpu@sha256:<digest>`.

---

## Docling Container

```ini
[Unit]
Description=Docling-serve (patched)
After=network-online.target

[Container]
ContainerName=docling
Image=docling.build
Network=rag.network
Environment=DOCLING_SERVE_ENABLE_UI=false
Environment=OMP_NUM_THREADS=2
Memory=3g

[Service]
Restart=always

[Install]
WantedBy=default.target
```

**Notable:** No `PublishPort`. Docling is intentionally not exposed on the host. It is reachable only from other containers on `rag.network`. This removes the earlier `5001:5001` mapping and the `127.0.0.1:5001` binding.

---

## Silent Partial Success

Docling can produce a "successful" conversion that contains:

- A few OCR'd characters (e.g., a text block "TUX").
- No `pictures`, no `drawing_file`, no `asset_dir`.

The document status says success but the VLM never saw the image. Always inspect `inputs/__parsed__/<name>.docling_raw/` (or `.parsed/` depending on version):

- `*.blocks.jsonl` first line: `drawing_file: true` and `asset_dir: true`.
- `*.drawings.json`: `llm_analyze_result.status: success`.

---

## OCR: Tesseract vs RapidOCR

| Engine | Prose | Monospaced terminal |
|---|---|---|
| RapidOCR (default) | Garbled: `Registered ports (1024-49151)` → `essdd sn sn :-nt d ns` | Also garbled |
| Tesseract (`eng`) | Clean prose | Garbled the other way: `tcp` → `tep`, `0.0.0.0:*` → `On0R0n 0s` |

Deployed `.env` uses Tesseract (`DOCLING_OCR_PRESET=tesseract`, `DOCLING_OCR_LANG=eng`). Trade-off: fixes prose at the cost of terminal output. Not fixable at the LightRAG layer — the text is corrupted before chunking. JSON extraction mode is more resilient to this corruption than delimiter-based extraction, but the underlying text loss remains.

**Tesseract quirk:** If `DOCLING_OCR_LANG` is unset, the container defaults to Spanish (`spa`) and Tesseract fails with `TesseractOcrCli has no model for the OCR language 'spa'`. Always set `DOCLING_OCR_LANG=eng`.

---

## Force OCR and the Native Text Layer

**Open question:** If `DOCLING_FORCE_OCR=true` (the default per `env.example`), Docling OCRs the page image even when the PDF has a native text layer. This could explain garbled text where the source PDF was fine.

**Untested:** Setting `force_ocr=false` on image-bearing PDFs (`name.[docling(force_ocr=false)].pdf`). The deployed `.env` does not set `DOCLING_FORCE_OCR`; the default `true` applies.

---

## Parse Cache

Docling parse results are cached in `inputs/__parsed__/<name>.docling_raw/`. Re-uploading an unchanged file uses the cache. Server-side Docling changes (like the patch) are invisible to the cache. To force a re-parse:

- Delete the document in the UI with "also delete file" ticked.
- Or set `LIGHTRAG_FORCE_REPARSE_DOCLING=true` in `.env`.

---

## Docling Container

```ini
[Unit]
Description=Docling-serve (patched)
After=network-online.target

[Container]
ContainerName=docling
Image=docling.build
Network=rag.network
Environment=DOCLING_SERVE_ENABLE_UI=false
Environment=OMP_NUM_THREADS=2
Memory=3g

[Service]
Restart=always

[Install]
WantedBy=default.target
```

**Notable:** No `PublishPort`. Docling is intentionally not exposed on the host. It is reachable only from other containers on `rag.network`.
