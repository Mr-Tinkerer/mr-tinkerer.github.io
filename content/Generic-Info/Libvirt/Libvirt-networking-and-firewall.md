---
title: Libvirt Networking, nftables, and UFW on Gigabyte Gaming A16 (CachyOS)
description: How libvirt, nftables, and UFW interact on the daily driver laptop, and two networking failures and their fixes.
tags:
  - Virtual_Machines
  - Networking
  - Firewall
  - Troubleshooting
---

# Libvirt Networking, nftables, and UFW

## Overview

This page documents how libvirt, nftables, and UFW interact on this host, and two networking failures that both came down to the same root cause: libvirt, nftables, and UFW each manage their own independent firewall state, and none is aware of the others' configuration.

## Division of Responsibilities

- **libvirt** creates and manages virtual bridge interfaces (`virbr0`, `virbr1`, `virbr2`, …), runs dnsmasq (DHCP/DNS) bound to each network's bridge, creates NAT (masquerade) rules for outbound guest traffic, and installs its own nftables rules directly into the kernel at runtime via netlink. It never reads from or writes to `/etc/nftables.conf`.
- **nftables** enforces host-wide firewall policy across the `input`, `forward`, and `output` hooks, independent of libvirt's own table.
- **UFW** is a policy front-end that also operates through the kernel's nftables (or iptables-nft) subsystem, but installs its own tables and base chains, independent of anything written by hand in `/etc/nftables.conf`. UFW has no knowledge of hand-written nftables rules, and vice versa.

This three-way independence is easy to overlook, since UFW is often assumed to be "the firewall," when in practice it's one of potentially several independent rule sets competing for the same hooks.

## libvirt Never Touches /etc/nftables.conf

A reasonable but incorrect assumption is that libvirt should update `/etc/nftables.conf` whenever a new virtual network is created. It doesn't: that file is static and only loaded at boot or on an explicit `nft -f`. libvirt instead inserts/removes rules directly in the live kernel ruleset over netlink as networks start and stop. Those rules are visible via:

```bash
sudo nft list ruleset
```

and never appear in `/etc/nftables.conf`, no matter how many networks are created. This distinction — static config file vs. live kernel ruleset — is the single most important thing to remember when diagnosing anything here.

| Location | Content | Updated by libvirt? |
|----------|---------|---------------------|
| `/etc/nftables.conf` | Static rules loaded at boot | No |
| `nft list ruleset` | Live, in-kernel rules currently enforced | Yes |

## Current Approach: Default-Accept Forward Policy with Targeted Denies

Originally, both failures were fixed using the same model: default-drop the `forward` hook, then allowlist only the interfaces known (`virbr*`). That model has a structural flaw:

- The rule is written against a snapshot of what's installed at the time, not against "traffic this host should generally be allowed to forward"
- Every subsequent forwarding-capable app (Docker, rootful Podman, Kubernetes/k3s, LXC/LXD, VirtualBox/VMware bridged networking, a routing VPN, etc.) is invisible to the rule until added manually
- The failure is silent: the new app's own firewall rules are entirely correct, but traffic is still dropped by this separate, independently-evaluated chain sharing the same hook

This happened twice: first hardcoded `virbr0`/`virbr1` names missed later libvirt networks, then after fixing that with a `virbr*` wildcard, the same allowlist model missed Docker entirely, because Docker didn't exist on the host when the rule was written.

### Replacement Model

Accept forwarding by default, deny only what's specifically wanted blocked:

```nft
table inet filter {
    chain forward {
        type filter hook forward priority filter; policy accept;

        # Explicit reject/drop rules go here only for traffic
        # specifically wanted blocked, e.g.:
        # iifname "eth0" oifname "wg0" drop comment "don't route WAN traffic onto the VPN"
    }
}
```

This inverts the default: any current or future forwarding-capable app works without needing a corresponding rule here. The chain's job shrinks to the small, deliberate set of forwarding paths specifically don't want.

### UFW Equivalent

The equivalent change for UFW's routed policy is to leave (or restore) it at its permissive default rather than tightening it and re-allowlisting `virbr+`:

```bash
sudo ufw default allow routed
sudo ufw reload
```

### Trade-off

This trades a smaller attack surface for operational reliability — this host will route traffic for any interface pair by default, including combinations not specifically reasoned about. For a general-purpose workstation running a shifting mix of VM/container tooling, that's the right call, since the realistic risk is "an app's traffic gets silently dropped and an hour is spent debugging it" (which happened twice already) rather than "an attacker pivots through unintended host forwarding." If ever repurposed as a router/gateway, the default-deny model (Sections 15/13) is still the more defensible starting point for that use case, maintained as a living allowlist.

### Verification

```bash
sudo nft list ruleset | grep -A5 "chain forward"
sudo ufw status verbose
```

Confirm the custom `inet filter forward` chain shows `policy accept`, and UFW's routed default reads `allow (routed)`, then test forwarding for each active subsystem independently.

## Reference Architecture

With the wildcard fix applied, VM traffic to the outside network follows this path:

```
Virtual Machine
      │
      ▼
virbrX (bridge interface, created and managed by libvirt)
      │
      ▼
libvirt NAT + dnsmasq (runtime nftables rules, table ip libvirt_network)
      │
      ▼
Host nftables forward chain (static policy layer, /etc/nftables.conf)
      │
      ▼
Internet
```

Two separate rule sets act on this traffic — libvirt's own dynamically-managed table, and the static policy in `/etc/nftables.conf` — and both must independently permit it.

## libvirt's Runtime NAT Table

Inspecting the live ruleset reveals a table never defined:

```nft
table ip libvirt_network
```

libvirt creates, populates, and maintains this table entirely on its own — guest NAT (masquerade) rules, subnet-specific routing, and per-network forwarding logic, one set per configured virtual network, generated automatically from that network's subnet.

This table is treated as read-only: it's not edited directly (libvirt may overwrite or conflict with changes), and its contents never show up in `/etc/nftables.conf`.

## Why Forwarding Can Break Even When libvirt is Configured Correctly

libvirt's own NAT config can be entirely correct while VM traffic still fails, because libvirt's table isn't the only one evaluated. The custom `forward` chain (in `/etc/nftables.conf`) is evaluated independently — if its policy is `drop` with no matching rule for the relevant bridge, it drops the packet before libvirt's NAT rules are ever relevant, regardless of how correctly libvirt is configured. This is the same "multiple independent rule sets, same hook" mechanism that reappears with UFW: a correctly configured ruleset doesn't guarantee traffic is permitted if another independently-evaluated ruleset can also reject it.

## Static nftables Config Supporting Dynamic libvirt Networking

### Forward Chain

Including the wildcard fix:

```nft
chain forward {
  type filter hook forward priority 0;
  policy drop;

  ct state established,related accept
  iifname "virbr*" accept
  oifname "virbr*" accept
}
```

(`ct state established,related accept` avoids re-evaluating return traffic for already-permitted connections — standard practice independent of libvirt.)

### Input Chain

DHCP/DNS requests from VMs are addressed to the host itself, so they must be explicitly permitted here:

```nft
iifname "virbr*" udp dport {53, 67} accept
```

Without this, VMs can be correctly bridged and NAT-configured yet still fail to get a DHCP lease, since the initial request never passes the host's own `input` filtering.

## Practices Now Avoided

| Practice | Consequence |
|----------|-------------|
| Hardcoding specific bridge names instead of wildcard matching | New virtual networks silently lose connectivity until the ruleset is manually updated |
| Manually editing `table ip libvirt_network` | Risk of conflicting with libvirt's internal state; changes may be reverted without notice |
| Assuming `/etc/nftables.conf` reflects libvirt-created rules | Leads to incorrect diagnosis, since the static file never shows libvirt's live NAT/routing rules |
| Assuming a single firewall tool (e.g. UFW) is the only ruleset in effect | Leads to failure when a second, independent ruleset silently overrides a correct one |

libvirt talks to the kernel exclusively over netlink rather than through config files — an intentional design choice, not an oversight. The live ruleset (`nft list ruleset`) is always checked, not just static config, when diagnosing connectivity issues.

## Failure 2: UFW Silently Blocking DHCP/DNS Traffic

### Symptom

With the static ruleset from the forward chain already in place — including the `input` rule for UDP 53/67 from `virbr*`, and a `forward` chain permitting routed traffic both ways — enabling UFW immediately broke DHCP lease acquisition for every VM, and consequently their internet access.

### Root Cause

UFW doesn't read or integrate with `/etc/nftables.conf` at all — it manages its own nftables table, with its own base chains independently hooked into `input`/`forward`/`output`. It was confirmed directly:

```bash
sudo nft list tables
```

```
table inet filter     # custom ruleset
table ip filter        # UFW's own, independently managed ruleset
```

Netfilter explicitly permits multiple independent base chains at the same hook, each with its own priority. A packet is evaluated against every base chain registered at that hook, in priority order; if any chain issues a terminating verdict (`drop`/`reject`), that verdict is final — the packet never reaches a later chain, even one that would have accepted it. So a correctly written `accept` rule in one table provides no protection if a different table evaluated earlier at the same hook rejects the packet first.

### Why UFW Specifically Rejected It

UFW's default incoming policy was `reject`, and UFW had no rule permitting ports 53/67 from `virbr*`, since these dynamically-created bridges are entirely unknown to UFW's own rule set. UFW's `input` chain dropped the DHCP/DNS packets before they ever reached the custom `inet filter` table's correct `accept` rule — the ruleset wasn't defective, it was simply never consulted.

## Diagnosing the Direction of the Failure: Input vs Forward

The first instinct given "VMs can't reach the internet" was to suspect an outbound (`forward`/`output`) restriction — for DHCP/DNS specifically, that instinct is wrong, and understanding why matters for diagnosing this class of failure quickly.

libvirt runs dnsmasq bound directly to each network's bridge (e.g., `virbr0`), acting as DHCP server and DNS resolver for guests on that network. From the host's netfilter hooks:

| Traffic | Destination | Netfilter hook |
|---------|-------------|----------------|
| DHCP/DNS request | dnsmasq process on the host | `input` |
| General internet traffic | External host, via NAT | `forward` |

A VM's DHCP request is addressed to a process running on the host itself — it terminates locally, making it `input` traffic. Only after a lease is obtained does general traffic become `forward` traffic.

In this case, `forward` was never actually broken — UFW's routed policy was already `allow`, and the `forward` chain already accepted `virbr*` traffic both ways. The VM never even reached the point of testing forwarded traffic, because it never got a DHCP lease in the first place.

### Confirming via Logs

With UFW logging enabled:

```bash
sudo journalctl -k --since "-2 min" | grep -i "UFW BLOCK"
```

```
... IN=virbr0 ... DPT=67 ...
```

`IN=` (not `OUT=`) plus a DHCP/DNS destination port confirms the block happened on the `input` hook. Reply traffic itself wasn't independently at risk — once the initial request is accepted, connection tracking classifies the reply as `established`/`related`, already accepted by both rulesets. The failure was entirely the initial inbound request being rejected before any reply was generated.

## Resolution

Since UFW's own chain — not the custom `inet filter` table — was dropping the traffic, it was fixed inside UFW:

```bash
sudo ufw allow in on virbr+ to any port 67 proto udp comment 'libvirt dhcp'
sudo ufw allow in on virbr+ to any port 53 comment 'libvirt dns'
sudo ufw reload
```

The trailing `+` is UFW's interface-name-prefix wildcard, covering `virbr0`, `virbr1`, `virbr2`, and any bridge libvirt creates in the future, without needing to revisit these rules as new networks appear.

### Verification

```bash
sudo ufw show added
sudo journalctl -k -f
```

After restarting a VM (or forcing a DHCP renewal inside one), look for no further `[UFW BLOCK]` entries on port 53/67, and confirm the VM gets a lease and working internet access.

### Caveat

These rules match on interface-name prefix specifically, not on "a libvirt-managed bridge" in general — any virtual network using a bridge name that doesn't match `virbr*` needs its own corresponding UFW rule.

## Consolidated Architecture, Including UFW

After the fix, two independent rule sets each separately have to permit VM traffic at every relevant hook:

```
Virtual Machine
      │
      ▼
virbrX (bridge interface, created by libvirt)
      │
      ▼
libvirt NAT + dnsmasq (runtime nftables rules, table ip libvirt_network)
      │
      ▼
──────────────── input hook (DHCP / DNS requests to host) ────────────────
   UFW's own chain        — must explicitly permit virbr+ : ports 53/67
   inet filter (custom)   — already permits virbr* : ports 53/67
   (a terminating verdict from either chain is final; both must accept)
      │
      ▼
──────────────── forward hook (routed internet traffic) ──────────────────
   UFW's own chain        — routed policy: allow
   inet filter (custom)   — already permits virbr* in both directions
      │
      ▼
Internet
```

*Note: the routed policy shown above (`allow`) is the permissive baseline that resolved Failure 2. A subsequent hardening step replaced this global allow with a scoped, `virbr+`-only exception, later superseded by the default-accept model.*

## Hardening (Superseded): Replacing UFW's Global Routed Policy with a Scoped Exception

Sections 9–12 left UFW's default forward policy at `allow (routed)` — sufficient to fix Failure 2, but broader than libvirt actually needs, since it permits forwarding between any pair of interfaces for any purpose.

Naively tightening this to `deny` also blocks the `virbr*` traffic VMs need. In practice this produced an asymmetric symptom: `ping 1.1.1.1` kept working from inside a guest, while DNS resolution and larger TCP flows (loading a web page) timed out — that asymmetry pointed at the routed policy itself as the new cause.

### The Targeted Fix Applied

```bash
# 1. Lock down the global forwarding policy
sudo ufw default deny routed

# 2. Allow selective routing only for libvirt's bridges, with a descriptive comment
sudo ufw route allow in on virbr+ comment 'libvirt internet routing'

# 3. Apply the changes
sudo ufw reload
```

### Resulting Config

```
Status: active
Logging: on (low)
Default: reject (incoming), allow (outgoing), deny (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
67/udp on virbr+            ALLOW IN    Anywhere                   # libvirt dhcp
53 on virbr+                ALLOW IN    Anywhere                   # libvirt dns
67/udp (v6) on virbr+       ALLOW IN    Anywhere (v6)              # libvirt dhcp
53 (v6) on virbr+           ALLOW IN    Anywhere (v6)              # libvirt dns

-- ROUTE --
To                         Action      From
--                         ------      ----
Anywhere on virbr+           ALLOW FWD   Anywhere                   # libvirt internet routing
```

> **Status: superseded, kept for reference.** This is the same allowlist-by-interface-name model as the wildcard fix, applied to UFW's routed policy. It has the same limitation: it silently blocks any other forwarding-capable app (Docker, Kubernetes, LXD, …) unless a matching `ufw route allow` rule is added too. The default-accept model is what's used now.

## Lessons Learned

1. libvirt manages firewall rules exclusively at runtime, via netlink — it never touches `/etc/nftables.conf`. The live ruleset is always checked, not just static config.
2. Rules referencing dynamically-created interfaces need wildcard matching (`virbr*` in nftables, `virbr+` in UFW) — hardcoded names silently fail for any interface created afterward.
3. Multiple independently-managed rule sets can be evaluated at the same netfilter hook — internal correctness in one doesn't guarantee traffic is permitted if another (UFW, firewalld, Docker, a hand-written table) also evaluates the same hook.
4. Diagnosis should follow where traffic is actually consumed, not the symptom's surface description — "can't reach the internet" doesn't say whether the failure is at `input` or `forward`.
5. Kernel logs (`IN=`/`OUT=` plus destination port) identify the responsible hook directly — these are checked before making config changes.
6. A global `allow (routed)` default is broader than libvirt requires, but tightening it to a `virbr*`-only allowlist doesn't scale to a host running an unknown/growing set of forwarding-capable apps — the default-accept model is what's used now.

## Failure 1: Hardcoded Bridge Interfaces

### The Problem

An early version of the `forward` chain enumerated bridges explicitly:

```nft
chain forward {
  policy drop;

  iifname virbr0 accept
  iifname virbr1 accept
}
```

At the time, only `virbr0` and `virbr1` existed, so this appeared to work.

### Why It Breaks

libvirt creates bridges dynamically as new virtual networks are defined/started (`virbr2`, `virbr3`, …). A ruleset hardcoding specific names can't account for interfaces that don't exist yet when it's written — any new network created afterward has its traffic silently dropped by the `drop` default, despite libvirt's own NAT/routing being entirely correct.

### The Fix — Wildcard Interface Matching

```nft
iifname "virbr*" accept
oifname "virbr*" accept
```

This matches `virbr0`, `virbr1`, `virbr2`, and any future bridge, without needing the ruleset edited again.

### General Principle

When writing firewall rules for a subsystem that manages its own dynamic set of interfaces (libvirt, Docker, etc.), match on the interface naming convention, not specific instances.

> **Warning: this chain's `policy drop` can silently break other kernel-forwarding applications.**
> The `chain forward { policy drop; ... }` block is a separate, independently-evaluated ruleset sitting at the same `forward` hook as every other app on the host that routes or bridges traffic through the kernel. Because its default policy is `drop` and it only allowlists `virbr*`, any other app whose traffic hits the `forward` hook and doesn't match `virbr*` gets silently dropped here — even with entirely correct rules of its own.
>
> This happened in practice: the rule was written to fix libvirt DHCP/DNS at a time when Docker wasn't in use, and later broke Docker container networking entirely once Docker was installed, since `docker0` and Docker's per-network bridges (`br-xxxxxxxxxxxx`) don't match `virbr*`.
>
> **Other apps/scenarios using the `forward` hook that could break the same way:** Docker/Podman in rootful bridge mode (rootless Podman usually routes through `slirp4netns`/`pasta` instead, so it's unaffected); Kubernetes/k3s/minikube/kind (CNI bridges — `flannel`, `cilium`, `calico`, `cni0`); LXC/LXD (`lxcbr0`, `lxdbr0`); VirtualBox/VMware host-only or bridged networking (`vboxnet*`); WireGuard/OpenVPN configured to route for other hosts; manual IP forwarding/NAT-gateway setups; systemd-nspawn containers with bridged networking.
>
> **The general lesson:** a rule set scoped to "only allow the interfaces currently cared about" with a `drop` default is a global policy decision, not a scoped one. Before adding a `policy drop` forward chain like this, audit what else on the host currently does (or might in the future do) kernel-level packet forwarding, and either widen the allowlist accordingly or prefer a less invasive default.
>
> **Status: superseded, kept for reference.** The wildcard fix above is still valid and still solves the immediate DHCP/DNS symptom, but the underlying default-drop-plus-allowlist model is no longer what's used, precisely because of the breakage described above — the default-accept model is what's used now. The wildcard-matching technique itself remains correct and is reused there.
