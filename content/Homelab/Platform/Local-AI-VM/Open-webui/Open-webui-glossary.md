---
title: Open WebUI Glossary
description: Terms used across the Open WebUI bucket.
tags:
  - glossary
---
## Open WebUI

A self-hosted, extensible web front-end for chatting with local or remote LLM backends (including llama.cpp and Ollama), with support for web search, tools, and RAG (retrieval-augmented generation). It's the piece I actually interact with day to day — [[Homelab/Platform/Local-AI-VM/Llama.cpp/Serving-as-a-service|llama-server]] does the raw inference, but Open WebUI supplies the ChatGPT-style browser interface, per-user chat history, model picker, and the settings that wire in [[Homelab/Platform/Local-AI-VM/Searxng/Searxng-setup|SearXNG]] for web search. Without it I'd be talking to llama.cpp over raw HTTP requests instead of a normal chat UI. See [Open WebUI (GitHub)](https://github.com/open-webui/open-webui).
