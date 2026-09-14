---
title: Collabora Office
description: Adding in-browser document editing to Nextcloud via a separate Collabora Online container.
tags:
  - Homelab
  - Nextcloud
---
[Nextcloud Office](https://docs.nextcloud.com/server/latest/admin_manual/office/installation.html) is powered by a separate [Collabora Online](https://docs.nextcloud.com/server/latest/admin_manual/office/example-docker.html) server, run here as its own container (`collabora`) on the same Podman network as the rest of the [[Homelab/Platform/Productivity-VM/Nextcloud/Nextcloud-installation|Nextcloud stack]], reachable at its own subdomain (`office.nextcloud.uhhhhh`).

Once running, the Nextcloud admin UI confirms the Collabora server is reachable and reports which URL each side is using to reach the other:

![[nextcloud-collabora-server-reachable-verification.png]]
*Nextcloud confirming Collabora Online is reachable, along with the URLs each side uses to reach the other.*

I confirmed document editing worked end-to-end by opening and editing a `.docx` file directly in the browser.

See [[Homelab/Platform/Productivity-VM/Nextcloud/Nextcloud-gotchas|Gotchas]] for the certificate-trust and Nginx-redirect problems this integration hit before it worked.
