---
title: Open WebUI Glossary
description: Terms used across the Open WebUI bucket.
tags:
  - glossary
---

## Overview

This page documents terms used across the Open WebUI bucket.

## Open WebUI

A self-hosted, extensible web front-end for chatting with local or remote LLM backends (including llama.cpp and Ollama), with support for web search, tools, and RAG (retrieval-augmented generation).

It supplies the ChatGPT-style browser interface, per-user chat history, model picker, and the settings that wire in SearXNG for web search. llama-server does the raw inference, but Open WebUI provides the interface for interactive use. Without it, raw HTTP requests are used instead of a normal chat UI.

See [Open WebUI (GitHub)](https://github.com/open-webui/open-webui).
