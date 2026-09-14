---
title: SearXNG Setup
description: Running a private SearXNG instance so local LLMs can search the web without depending on a third-party search API.
tags:
  - Homelab
  - Searxng
---
[SearXNG](https://github.com/searxng/searxng) runs as a [[Homelab/Logical/Containerization/Tool-choices|Podman]] container on the [[Homelab/Logical/Virtual-machines/Vm-allocation|Local AI VM]], giving [[Homelab/Platform/Local-AI-VM/Open-webui/Open-webui-setup|Open WebUI]] a private, self-hosted search backend instead of a third-party search API. Reachable at `search.uhhhhh` through [[Homelab/Software/Nginx/Reverse-proxy-setup|Nginx]] (see [search.conf](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Nginx/conf.d/search.conf)). The maintained quadlet is kept in the infrastructure repo: [searxng.container](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/Local-AI%20VM/Quadlets/searxng.container).

Unlike Open WebUI's `Network=host` setup, the SearXNG quadlet publishes a mapped port (`PublishPort=8888:8080` — host port 8888 to the container's internal port 8080) rather than sharing the host network namespace directly. It persists two separate volumes: `%h/searxng/config/:/etc/searxng` (SearXNG's own configuration, e.g. `settings.yml`) and `%h/searxng/data/:/var/cache/searxng` (its cache). [[Homelab/Platform/Local-AI-VM/Open-webui/Open-webui-setup|Open WebUI's]] quadlet declares a hard `Requires=`/`After=` dependency on this service, so SearXNG must be up before Open WebUI starts.

See [[Homelab/Platform/Local-AI-VM/Open-webui/Open-webui-setup#Web search via SearXNG|Open WebUI]] for how this instance is wired into the chat front-end.
