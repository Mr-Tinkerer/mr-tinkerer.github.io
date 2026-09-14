---
title: Nextcloud Gotchas
description: Certificate trust, security warnings, and a stray Nginx redirect hit while getting Nextcloud and Collabora working together.
tags:
  - Homelab
  - Nextcloud
  - Gotchas
---
# Security & setup warnings after first install

Nextcloud's admin UI flagged several errors and warnings after the initial install, before HTTPS and reverse-proxy trust were configured:

![[nextcloud-security-setup-warnings-before-fix.png]]
*Nextcloud's "Security & setup warnings" panel flagging incorrect forwarded-for headers and insecure HTTP access, before Nginx trust and HTTPS-only were configured.*

**Fix:** I told Nextcloud to trust the Nginx reverse proxy and its subnet (`trusted_proxies`), and to always assume HTTPS (`overwriteprotocol`). I baked these settings into the custom Nextcloud image so they survive container rebuilds.

# Collabora and Nextcloud couldn't validate each other's certificates

Both Nextcloud and Collabora need to reach each other over HTTPS, but neither trusted the other's certificate since both use this home lab's private CA rather than a public one (see [[Homelab/Software/Nginx/Ssl-certificates|SSL certificates]]). I had Claude help me debug this, and my fix was to manually install the private CA's root certificate inside both the `nextcloud-custom` and `collabora-custom` container images, rather than disabling certificate verification.

Along the way, a red herring: Claude flagged that the Collabora container appeared to fail spawning a child process, which looked like a Podman-specific security restriction (its rootless/more restrictive defaults compared to Docker) — I swapped to Docker briefly to test this theory, which didn't actually fix it, since the real cause turned out to be unrelated (see below).

# The actual cause: a stray Nginx redirect

The real cause of the Collabora connection failures was a copy-paste mistake in Nginx's config for the Collabora subdomain (`office.nextcloud.conf`'s HTTP→HTTPS block was redirecting to `http://` instead of `https://`) — since fixed. See [[Homelab/Software/Nginx/Reverse-proxy-setup|the Nginx bucket]] for this service's reverse-proxy config in general.

# Mobile app certificate trust

The Nextcloud mobile app did not work with the private CA out of the box (root cause not further investigated). See [[Homelab/Software/Nginx/Ssl-certificates#Mobile browser certificate trust|SSL certificates]] for a related mobile-browser certificate-trust issue (Waterfox), which is a Nginx/CA-level issue rather than a Nextcloud-specific one.
