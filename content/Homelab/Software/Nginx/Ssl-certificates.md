---
title: Nginx SSL Certificates
description: Standing up a private certificate authority so HTTPS works for the home lab's internal-only domain, without public-CA-signed certs.
tags:
  - Homelab
  - Nginx
  - SSL
---

## Overview

This page covers standing up a private certificate authority so HTTPS works for the home lab's internal-only domain, without public-CA-signed certs.

## Why a Private CA

HTTPS needs a signed certificate. There are three ways to get one:

- **Self-signed** — Free, but untrusted by every client by default, requiring manual per-browser approval
- **Public CA** — Trusted automatically almost everywhere, but requires a real registered domain. Not usable here since the home lab uses an unregistered, custom TLD
- **Private CA** — Same trust mechanics as a public CA, but only trusted on devices where its root certificate is manually installed. This was the option chosen

The steps below follow a guide for a local private CA (written for Ubuntu, but the underlying `openssl` commands are distro-agnostic and worked unchanged on OpenWrt after the `openssl-util` package was installed).

## Creating the Root CA

```bash
openssl genrsa -des3 -out openCA.key 2048
openssl req -x509 -new -nodes -key openCA.key -sha256 -days 1825 -out openCA.pem
```

This produces `openCA.key` (the CA's private key, passphrase-protected) and `openCA.pem` (the root certificate to distribute to clients).

## Trusting the Root Certificate on Clients

The root certificate (`openCA.pem`) needs to be installed on every client that should trust it without a browser warning. The exact steps are device/OS-specific, so these are what worked on the specific devices tested, not a universal procedure:

- **Linux (CachyOS/Arch Linux, with `ca-certificates` + `p11-kit` installed):** `trust anchor --store openCA.pem`
- **Android (Samsung S24):** `Settings → Security and Privacy → More Security Settings → Install From Phone Storage → CA certificate`
- **Windows 11:** Import via the Microsoft Management Console's Certificates snap-in, under `Certificates (Local Computer) → Third-Party Root Certificate Authorities`. This import didn't propagate to Firefox on Windows — Firefox needed the certificate imported separately, directly into its own certificate store.

### Mobile Browser Certificate Trust

On the Samsung S24, the Waterfox browser stopped trusting the private CA's certificate at some point despite the CA being installed at the OS level above. Re-enabling `security.enterprise_roots.enabled` in `about:config` restored trust by telling Waterfox to actually consult the OS-level certificate store instead of only its own bundled list.

## Issuing a Certificate for a Domain

```bash
openssl genrsa -out openwrt.key 2048
openssl req -new -key openwrt.key -out openwrt.csr
```

An extension config file (`openwrt.ext`) supplies the Subject Alternative Name the certificate needs:

```toml
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = openwrt.uhhhhh
```

Then the CA signs the request:

```bash
openssl x509 -req -in openwrt.csr -CA openCA.pem -CAkey openCA.key -CAcreateserial \
  -out openwrt.crt -days 825 -sha256 -extfile openwrt.ext
```

Once Nginx is pointed at the resulting `.crt`/`.key` pair and restarted, HTTPS works with no browser trust warning, provided the root CA has already been installed on the client.

See [trusted HTTPS screenshot](Images/nginx-https-trusted-no-warning-private-ca.png) for HTTPS working with no certificate warning, after the private CA's root certificate was installed on the client and a per-domain cert was signed with it.

## Repeating This Per Domain

A small maintained shell script was written that wraps the per-domain steps above (key, CSR, extension file, signing) so a new domain's certificate can be generated with one command. It was modified from the original guide's version to fit this home lab's paths, and is linked to rather than pasted, since it's an actively maintained script: [make_cert.sh](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/make_cert.sh).
