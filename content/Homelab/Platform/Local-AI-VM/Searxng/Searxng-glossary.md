---
title: SearXNG Glossary
description: Terms used across the SearXNG bucket.
tags:
  - glossary
---
## SearXNG

A free, self-hostable metasearch engine that aggregates results from other search engines (Google, Bing, DuckDuckGo, etc.) into one result set, without tracking the person searching or forwarding queries to those engines under their own identity. I run it here specifically so [[Homelab/Platform/Local-AI-VM/Open-webui/Open-webui-setup|Open WebUI]] has a private web-search backend for local models, instead of depending on a third-party search API (with its own cost, rate limits, and data-sharing terms). It's what actually goes out and fetches live results when I ask a local model something that needs current information rather than just what's in its training data. See [SearXNG (official site)](https://docs.searxng.org/).
