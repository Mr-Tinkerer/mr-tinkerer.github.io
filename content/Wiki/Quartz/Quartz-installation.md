---
title: Quartz Installation
description: Getting Quartz cloned, initialized, and its plugins installed.
tags:
  - wiki
  - quartz
---

I installed [[Quartz-glossary#Quartz|Quartz]] with the following commands:

```bash
git clone https://github.com/jackyzha0/quartz.git
cd quartz
npm i
npx quartz create
```

`npx quartz create` asks a few setup questions. I answered:

- **Template:** `obsidian`
- **Content strategy:** empty folder (I'm populating it with my own vault, not Quartz's sample content)
- **Base URL:** `mr-tinkerer.github.io`

Once the site was initialized, I refreshed and installed [[Quartz-glossary#Plugin (Quartz)|plugins]]:

```bash
npx quartz plugin install --latest
npx quartz plugin install --from-config
```

- `--latest` bumps every installed plugin to its latest version.
- `--from-config` installs whatever plugins are listed in `quartz.config.yaml` but not yet present locally.

See [[Quartz-configuration|Configuration]] for what I changed after this initial install.
