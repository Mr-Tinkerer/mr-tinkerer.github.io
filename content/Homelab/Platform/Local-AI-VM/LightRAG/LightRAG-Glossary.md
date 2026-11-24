---
title: LightRAG and Docling Glossary
description: Terms specific to the LightRAG and Docling RAG stack on the Local-AI-VM Fedora platform.
tags:
  - LightRAG
  - Docling
  - homelab
  - fedora
  - rag
  - glossary
---

Terms that are specific to the LightRAG and Docling RAG stack. General llama.cpp/LLM vocabulary that applies across all Llama.cpp instances is defined once in [[Homelab/Platform/Local-AI-VM/Llama.cpp/Llama-cpp-glossary|the canonical Llama.cpp glossary]] — link there instead of redefining it here.

---

## LightRAG

LightRAG is a self-hosted RAG (Retrieval-Augmented Generation) server that combines knowledge graph extraction with vector embedding for improved retrieval quality. It ingests documents, extracts entities and relations to build a knowledge graph, and embeds chunks for vector search.

### RAG Retrieval Modes

LightRAG supports multiple retrieval modes:

- **naive:** Uses only vector chunks, no graph traversal
- **local:** Uses knowledge graph for retrieval
- **global:** Uses global knowledge graph for retrieval
- **hybrid:** Combines graph and vector retrieval
- **mix:** Uses both graph and vector retrieval (WebUI default, see [[Homelab/Platform/Local-AI-VM/LightRAG/LightRAG-Parser-Routing#Retrieval Modes|LightRAG Stack]])

---

## Docling

Docling is a document parsing library that converts PDFs and other document formats into structured data (blocks, drawings, tables). It uses layout detection to identify different content types and can route documents to different processing engines.

### Docling Engines

- **docling:** Main parsing engine with layout detection and OCR support
- **native:** Simple text extraction for Markdown and similar formats
- **legacy:** Fallback parser for unsupported formats

---

## Parser Routing

LightRAG uses the `LIGHTRAG_PARSER` environment variable to route different file types to different processing engines with specific options. Rules are evaluated left to right; the first match wins.

### Routing Options

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

### The `!` Option

The `!` option applies at the extraction stage. It skips entity extraction, relation extraction, entity merge, relation merge, and graph writes for the affected document, but still runs parsing, VLM analysis (if `i` is set), chunking, and embedding. This is useful for image documents where you want vector indexing without polluting the knowledge graph with hallucinated entities.

---

## Entity Deduplication

LightRAG's built-in entity auto-merge (`ENABLE_ENTITY_AUTO_MERGE=true`) is non-functional in the tested version. The post-ingestion `dedupe_entities.py` script is used instead.

### Deduplication Stages

1. **Rule-based normalization (auto-merge):** Case folding, token reordering, acronym-aware singularization
2. **Morphological equivalence:** Detects variations like "Prepare" / "Preparation Phase"
3. **Explicit alias file:** JSON array of [a, b] pairs to merge unconditionally
4. **Embedding similarity:** Cosine similarity of entity-name embeddings
5. **Two-way LLM verification:** For gray pairs, asks the LLM twice (A, B and B, A); both must say yes
6. **Cannot-link-aware merge application:** Checks that merges don't create conflicts with reject pairs
7. **Apply merges to LightRAG:** POST `/graph/entities/merge` for each component with >1 member

See [[Homelab/Platform/Local-AI-VM/LightRAG/Entity-Deduplication|Entity Deduplication]] for full details.

### FP/FN Asymmetry

For entity resolution:
- **FP (false positive):** Merging two distinct concepts. Destroys graph structure — edges and chunk references absorbed into the wrong node. Hard to undo.
- **FN (false negative):** Leaving a duplicate. Recoverable with a second pass.

**FP is strictly worse than FN.** A model with higher accuracy but more FPs is a worse choice.

---

## VLM (Vision Language Model)

A multimodal model that can analyze images and generate text descriptions. In the LightRAG stack, the VLM is used to analyze images extracted from documents and generate descriptive text that can be embedded and retrieved.

### VLM Processing

When enabled (via the `i` routing option), the VLM analyzes images and produces:
- A textual description of the image content
- This description is chunked and embedded for vector retrieval

The VLM description chunk is retrievable by vector search but not by graph traversal.

---

## JSON Extraction Mode

LightRAG supports two extraction formats: delimiter-based and JSON. JSON mode (`ENTITY_EXTRACTION_USE_JSON=true`) is more robust and structured, reducing format-leak failures where the model copies prompt placeholders into entity names.

### Token Budget Interaction

JSON output is more verbose. With `OPENAI_LLM_MAX_TOKENS=3072`, chunks may truncate. Raising to 5120 fixed this issue. Both `OPENAI_LLM_MAX_TOKENS` and `OPENAI_LLM_MAX_COMPLETION_TOKENS` should be set to 5120 for safety.

---

## Thinking versus Non-Thinking

Some models support both thinking (chain-of-thought) and non-thinking modes. For structured extraction tasks like entity deduplication, thinking mode is incompatible with single-token output tasks — the reasoning block consumes all tokens before any output is produced.

### Model Selection for Extraction

| Model | Verdict |
|---|---|
| Qwen3.5-4B thinking | Incompatible (content: "" on all responses) |
| Qwen3.5-4B thinkless | **Selected** - Clean JSON output |
| Gemma-3-4B | Weak; format leaks; hallucinates on empty chunks |
| Llama-3.1-8B | Cleaner graph but OOMs on 8 GB when paired with a 7B VLM |

---

## Quadlet Deployment

Quadlet is Podman's systemd-like integration for managing containers. It allows defining build and run units in `~/.config/containers/systemd/` that are managed by `systemd --user`.

### Quadlet Units

- **docling.build:** Build unit for the patched Docling image
- **docling.container:** Docling service
- **lightrag.container:** LightRAG service
- **rag.network:** Bridge network for container communication

### Quadlet Quirks

1. **Transient units:** `systemctl --user enable` does not work; use `[Install] WantedBy=default.target` plus `loginctl enable-linger ai`
2. **depends_on → Requires= + After=:** Start ordering only, not readiness
3. **Rootless memory limits:** If cgroup memory delegation is not enabled, limits are silently ignored
4. **Absolute paths only:** Quadlet does not resolve `./`

---

## Known Discrepancies

### llama-server Binding vs. .env Address

- `llama-server.service` active line: `--host 127.0.0.1 --port 9931` (loopback only)
- `.env` addresses: `http://host.containers.internal:9931/v1`

`host.containers.internal` in rootless Podman resolves to the container's bridge gateway, not the host's `127.0.0.1`. A container on `rag.network` cannot reach `127.0.0.1:9931` on the host. This is an open item that needs verification.

### LightRAG-* Role Presets

The uploaded `presets.ini` has the `[LightRAG-Logic]`, `[LightRAG-Vision]`, `[LightRAG-Embedding]` sections commented out. The deployed `.env` uses those exact role names as the model strings. If the uploaded `presets.ini` is the live one, requests for `LightRAG-Logic` will 404.

---

## Archived Status

The RAG stack was archived (Era F) after the realization that the original goal — a quiz agent over an Obsidian notes vault — was better served by direct `.md` file access than by the full RAG pipeline. The deployed architecture remains in place but is not the active focus. The Quizzer preset in `presets.ini` (Qwen3.5-2B, ctx 32768) is the post-RAG quiz agent's model.
