---
title: SSL certificates
description: Standing up a private certificate authority so HTTPS works for the home lab's internal-only domain, without public-CA-signed certs.
tags:
  - Homelab
  - Nginx
  - SSL
---
# Why a private CA

HTTPS needs a signed certificate. There are three ways to get one:

- **Self-signed** — free, but untrusted by every client by default, requiring manual per-browser approval.
- **Public [[Homelab/Software/Nginx/Nginx-glossary#Certificate authority (CA)|CA]]** — trusted automatically almost everywhere, but requires a real registered domain. Not usable here since the home lab uses an unregistered, custom TLD (see [[Homelab/Logical/Home-network/Dns-naming-scheme|the DNS naming scheme]]).
- **Private CA** — same trust mechanics as a public CA, but only trusted on devices where its root certificate is manually installed. This is the option I chose.

The steps below follow [this guide for a local private CA](https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/) (written for Ubuntu, but the underlying `openssl` commands are distro-agnostic and worked unchanged on OpenWrt after I installed the `openssl-util` package).

# Creating the root CA

```bash
openssl genrsa -des3 -out openCA.key 2048
openssl req -x509 -new -nodes -key openCA.key -sha256 -days 1825 -out openCA.pem
```

This produces `openCA.key` (the CA's private key, passphrase-protected) and `openCA.pem` (the root certificate to distribute to clients).

# Trusting the root certificate on clients

I need to install the root certificate (`openCA.pem`) on every client that should trust it without a browser warning. The exact steps are device/OS-specific, so treat the ones below as what worked on the specific devices I tested, not a universal procedure:

- **Linux (CachyOS/Arch Linux, with `ca-certificates` + `p11-kit` installed):** `trust anchor --store openCA.pem`
- **Android (Samsung S24):** `Settings → Security and Privacy → More Security Settings → Install From Phone Storage → CA certificate`
- **Windows 11:** import via the Microsoft Management Console's Certificates snap-in, under `Certificates (Local Computer) → Third-Party Root Certificate Authorities`. This import did not propagate to Firefox on Windows — Firefox needed the certificate imported separately, directly into its own certificate store.

## Mobile browser certificate trust

On the Samsung S24, the Waterfox browser stopped trusting the private CA's certificate at some point despite my having installed the CA at the OS level above; re-enabling `security.enterprise_roots.enabled` in `about:config` restored trust by telling Waterfox to actually consult the OS-level certificate store instead of only its own bundled list.

# Issuing a certificate for a domain

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

Then I have the CA sign the request:

```bash
openssl x509 -req -in openwrt.csr -CA openCA.pem -CAkey openCA.key -CAcreateserial \
  -out openwrt.crt -days 825 -sha256 -extfile openwrt.ext
```

Once I point Nginx at the resulting `.crt`/`.key` pair and restart it, I get HTTPS with no browser trust warning, provided the root CA has already been installed on the client.

![[nginx-https-trusted-no-warning-private-ca.png]]
*HTTPS working with no certificate warning, after I installed the private CA's root certificate on the client and signed a per-domain cert with it.*

# Repeating this per domain

I wrote a small maintained shell script that wraps the per-domain steps above (key, CSR, extension file, signing) so I can generate a new domain's certificate with one command. I modified it from the original guide's version to fit this home lab's paths, and I'm linking to it here rather than pasting it, since it's an actively maintained script: [make_cert.sh](https://github.com/Mr-Tinkerer/Project-Dump/blob/main/Homelab/make_cert.sh).
