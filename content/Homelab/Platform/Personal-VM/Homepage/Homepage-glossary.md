---
title: Homepage Glossary
description: Terms used across the Homepage bucket.
tags:
  - glossary
---

## Overview

This page documents terms used across the Homepage bucket.

## Homepage

A fast, self-hosted "start page" dashboard with widgets/integrations for many self-hosted services, configured through YAML files rather than a database-backed admin UI.

It serves as the single bookmarkable landing page for the whole home lab. Since every service sits behind the home lab's own unregistered TLD, typing a bare hostname into a browser often gets treated as a search query instead of a URL, so Homepage provides one link-of-links to bookmark instead of remembering/typing every service's full address.

Widgets pull live status/data from several services (e.g., Nextcloud, OpenWrt, Proxmox) directly into the dashboard.

See [Homepage (official site)](https://gethomepage.dev/).

## SELinux

A Linux kernel security module enforcing mandatory access control policies, enabled and enforcing by default on openSUSE and other security-focused distros. Files/processes carry security context labels, and access is denied if the acting process's label isn't permitted — including a container process trying to write to a bind-mounted host directory, unless the mount is relabeled for it.

See [SELinux (Wikipedia)](https://en.wikipedia.org/wiki/Security-Enhanced_Linux).

## firewalld

A dynamic firewall management daemon used by default on openSUSE, Fedora, and RHEL-family distros, managing `nftables`/`iptables` rules through named zones.

See [firewalld (official site)](https://firewalld.org/).
