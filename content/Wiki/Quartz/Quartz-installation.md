---
title: Quartz Installation on Mr.Tinkerer's Wiki
description: Getting Quartz cloned, initialized, and plugins installed.
tags:
  - wiki
  - quartz
---

## Overview

This guide covers Quartz installation and initialization, including plugin setup and configuration.

## Prerequisites

* Git installed and configured
* Node.js and npm installed
* Access to the Quartz GitHub repository

## Installation Steps

### Clone and Initialize

Clone the Quartz repository and install dependencies:

```bash
git clone https://github.com/jackyzha0/quartz.git
cd quartz
npm i
```

### Configuration

Run the Quartz creation wizard:

```bash
npx quartz create
```

Answer the following setup questions:

- **Template:** `obsidian`
- **Content strategy:** empty folder (populating with own vault rather than sample content)
- **Base URL:** `mr-tinkerer.github.io`

### Plugin Installation

Install and update plugins to their latest versions:

```bash
npx quartz plugin install --latest
npx quartz plugin install --from-config
```

- `--latest` bumps every installed plugin to its latest version
- `--from-config` installs plugins listed in `quartz.config.yaml` that are not yet present locally

## Verification

Verify installation by running the Quartz development server and confirming the site loads successfully.

See [[Wiki/Quartz/Quartz-configuration|Configuration]] for post-installation modifications.
