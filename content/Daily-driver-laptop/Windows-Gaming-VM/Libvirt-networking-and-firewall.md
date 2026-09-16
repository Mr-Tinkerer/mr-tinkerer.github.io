---
title: Libvirt Networking, nftables, and UFW
description: How libvirt, nftables, and UFW interact on this host, and two networking failures I hit and fixed.
tags:
  - Virtual_Machines
  - Networking
  - Firewall
  - Troubleshooting
---
# Libvirt Networking, nftables, and UFW

I hit two separate VM networking failures on this host that both came down to the same root cause: libvirt, nftables, and UFW each manage their own independent firewall state, and none of them is aware of the others' configuration. This page records how they interact, why the failures happened, and how I fixed them — so I can diagnose the same class of problem quickly if it recurs.

---

## 1. Division of responsibilities

- **libvirt** creates and manages virtual bridge interfaces (`virbr0`, `virbr1`, `virbr2`, …), runs `dnsmasq` (DHCP/DNS) bound to each network's bridge, creates NAT (masquerade) rules for outbound guest traffic, and installs its own nftables rules directly into the kernel at runtime via netlink. It never reads from or writes to `/etc/nftables.conf`.
- **nftables** enforces host-wide firewall policy across the `input`, `forward`, and `output` hooks, independent of libvirt's own table.
- **UFW** is a policy front-end that also operates through the kernel's nftables (or iptables-nft) subsystem, but installs its **own** tables and base chains, independent of anything I write by hand in `/etc/nftables.conf`. UFW has no knowledge of hand-written nftables rules, and vice versa.

This three-way independence is easy to overlook, since UFW is often assumed to be "the firewall," when in practice it's one of potentially several independent rule sets competing for the same hooks.

---

# 2. libvirt network modes
| Libvirt Networking Mode | Description                                                     | Best For                         |
| ----------------------- | --------------------------------------------------------------- | -------------------------------- |
| **NAT**                 | Simple, default-style networking with outbound internet access  | General-purpose VMs              |
| **Routed**              | VMs use routed IP addresses without traditional NAT             | Servers and routable VM networks |
| **Open**                | Host bridge with minimal libvirt filtering                      | Advanced/custom networking       |
| *Isolated**             | VMs can communicate with each other but have no external access | Testing and private networks     |
| **SR-IOV**              | Direct NIC virtual functions for near-native performance        | High-performance workloads       |

## 3. libvirt never touches `/etc/nftables.conf`

A reasonable but incorrect assumption is that libvirt should update `/etc/nftables.conf` whenever I create a new virtual network. It doesn't: that file is static and only loaded at boot or on an explicit `nft -f`. libvirt instead inserts/removes rules directly in the live kernel ruleset over netlink as networks start and stop. Those rules are visible via:

```bash
sudo nft list ruleset
```

and never appear in `/etc/nftables.conf`, no matter how many networks I create. This distinction — static config file vs. live kernel ruleset — is the single most important thing to remember when diagnosing anything here.

| Location | Content | Updated by libvirt? |
|---|---|---|
| `/etc/nftables.conf` | Static rules loaded at boot | No |
| `nft list ruleset` | Live, in-kernel rules currently enforced | Yes |

---

## 4. My current approach: default-accept forward policy with targeted denies

I originally fixed both failures below (Sections 9 and 15) using the same model: default-drop the `forward` hook, then allowlist only the interfaces I knew about (`virbr*`). That model has a structural flaw I didn't see until a second forwarding-capable application showed up:

- The rule is written against a *snapshot* of what's installed at the time, not against "traffic this host should generally be allowed to forward."
- Every subsequent forwarding-capable app (Docker, rootful Podman, Kubernetes/k3s, LXC/LXD, VirtualBox/VMware bridged networking, a routing VPN, etc.) is invisible to the rule until I remember to add it.
- The failure is silent: the new app's own firewall rules are entirely correct, but traffic is still dropped by this separate, independently-evaluated chain sharing the same hook.

This is exactly what happened twice on my host: first hardcoded `virbr0`/`virbr1` names missed later libvirt networks, then after fixing that with a `virbr*` wildcard, the same allowlist model missed Docker entirely, because Docker didn't exist on the host when I wrote the rule.

**The replacement model I use now:** accept forwarding by default, deny only what I specifically want blocked:

```nft
table inet filter {
	chain forward {
		type filter hook forward priority filter; policy accept;

		# Explicit reject/drop rules go here only for traffic
		# I specifically want to block, e.g.:
		# iifname "eth0" oifname "wg0" drop comment "don't route WAN traffic onto the VPN"
	}
}
```

This inverts the default: any current or future forwarding-capable app works without needing a corresponding rule here. The chain's job shrinks to the small, deliberate set of forwarding paths I specifically don't want.

The equivalent change for UFW's routed policy (see Section 13) is to leave (or restore) it at its permissive default rather than tightening it and re-allowlisting `virbr+`:

```bash
sudo ufw default allow routed
sudo ufw reload
```

**Trade-off I'm accepting:** this trades a smaller attack surface for operational reliability — this host will route traffic for any interface pair by default, including combinations I haven't specifically reasoned about. For a general-purpose workstation running a shifting mix of VM/container tooling, that's the right call for me, since the realistic risk is "an app's traffic gets silently dropped and I spend an hour debugging it" (which happened twice already) rather than "an attacker pivots through unintended host forwarding." If I ever repurpose this host as a router/gateway, the default-deny model (Sections 15/13) is still the more defensible starting point for that use case, maintained as a living allowlist.

**Verification:**
```bash
sudo nft list ruleset | grep -A5 "chain forward"
sudo ufw status verbose
```
I confirm the custom `inet filter forward` chain shows `policy accept`, and UFW's routed default reads `allow (routed)`, then test forwarding for each active subsystem independently.

---

## 5. Reference architecture

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

Two separate rule sets act on this traffic — libvirt's own dynamically-managed table, and my static policy in `/etc/nftables.conf` — and both must independently permit it.

---

## 6. libvirt's runtime NAT table

Inspecting the live ruleset reveals a table I never defined myself:

```nft
table ip libvirt_network
```

libvirt creates, populates, and maintains this table entirely on its own — guest NAT (masquerade) rules, subnet-specific routing, and per-network forwarding logic, one set per configured virtual network, generated automatically from that network's subnet.

I treat this table as read-only: I don't edit it directly (libvirt may overwrite or conflict with my changes), and I don't expect its contents to ever show up in `/etc/nftables.conf` (Section 2).

---

## 7. Why forwarding can break even when libvirt is configured correctly

libvirt's own NAT config can be entirely correct while VM traffic still fails, because libvirt's table isn't the only one evaluated. My own `forward` chain (in `/etc/nftables.conf`) is evaluated independently — if its policy is `drop` with no matching rule for the relevant bridge, it drops the packet before libvirt's NAT rules are ever relevant, regardless of how correctly libvirt is configured. This is the same "multiple independent rule sets, same hook" mechanism that reappears with UFW in Section 9 — a correctly configured ruleset doesn't guarantee traffic is permitted if another independently-evaluated ruleset can also reject it.

---

## 8. Static nftables config supporting dynamic libvirt networking

**Forward chain**, including the wildcard fix from Section 15:
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

**Input chain** — DHCP/DNS requests from VMs are addressed to the host itself (see Section 10 for the input vs. forward distinction), so they must be explicitly permitted here:
```nft
iifname "virbr*" udp dport {53, 67} accept
```
Without this, VMs can be correctly bridged and NAT-configured yet still fail to get a DHCP lease, since the initial request never passes the host's own `input` filtering.

---

## 9. Practices I now avoid

| Practice | Consequence |
|---|---|
| Hardcoding specific bridge names instead of wildcard matching | New virtual networks silently lose connectivity until I manually update the ruleset |
| Manually editing `table ip libvirt_network` | Risk of conflicting with libvirt's internal state; changes may be reverted without notice |
| Assuming `/etc/nftables.conf` reflects libvirt-created rules | Leads to incorrect diagnosis, since the static file never shows libvirt's live NAT/routing rules |
| Assuming a single firewall tool (e.g. UFW) is the only ruleset in effect | Leads to the failure in Section 9, where a second, independent ruleset silently overrides a correct one |

libvirt talks to the kernel exclusively over netlink rather than through config files — an intentional design choice, not an oversight. I always check the live ruleset (`nft list ruleset`), not just static config, when diagnosing connectivity issues.

---

## 10. Failure 2: UFW silently blocking DHCP/DNS traffic

**Symptom:** with the static ruleset from Section 7 already in place — including the `input` rule for UDP 53/67 from `virbr*`, and a `forward` chain permitting routed traffic both ways — enabling UFW immediately broke DHCP lease acquisition for every VM, and consequently their internet access.

**Root cause:** UFW doesn't read or integrate with `/etc/nftables.conf` at all — it manages its own nftables table, with its own base chains independently hooked into `input`/`forward`/`output`. I confirmed this directly:
```bash
sudo nft list tables
```
```
table inet filter     # my custom ruleset (Section 7)
table ip filter        # UFW's own, independently managed ruleset
```
Netfilter explicitly permits multiple independent base chains at the same hook, each with its own priority. A packet is evaluated against every base chain registered at that hook, in priority order; if **any** chain issues a terminating verdict (`drop`/`reject`), that verdict is final — the packet never reaches a later chain, even one that would have accepted it. So a correctly written `accept` rule in one table provides no protection if a *different* table evaluated earlier at the same hook rejects the packet first.

**Why UFW specifically rejected it:** UFW's default incoming policy was `reject`, and UFW had no rule permitting ports 53/67 from `virbr*`, since these dynamically-created bridges are entirely unknown to UFW's own rule set. UFW's `input` chain dropped the DHCP/DNS packets before they ever reached my `inet filter` table's correct `accept` rule — my ruleset wasn't defective, it was simply never consulted.

---

## 11. Diagnosing the direction of the failure: input vs. forward

My first instinct given "VMs can't reach the internet" was to suspect an outbound (`forward`/`output`) restriction — for DHCP/DNS specifically, that instinct is wrong, and understanding why matters for diagnosing this class of failure quickly.

libvirt runs `dnsmasq` bound directly to each network's bridge (e.g. `virbr0`), acting as DHCP server and DNS resolver for guests on that network. From the host's netfilter hooks:

| Traffic | Destination | Netfilter hook |
|---|---|---|
| DHCP/DNS request | dnsmasq process on the host | `input` |
| General internet traffic | External host, via NAT | `forward` |

A VM's DHCP request is addressed to a process running on the host itself — it terminates locally, making it `input` traffic. Only after a lease is obtained does general traffic become `forward` traffic.

In my case, `forward` was never actually broken — UFW's routed policy was already `allow`, and my `forward` chain already accepted `virbr*` traffic both ways. The VM never even reached the point of testing forwarded traffic, because it never got a DHCP lease in the first place.

**Confirming via logs**, with UFW logging enabled:
```bash
sudo journalctl -k --since "-2 min" | grep -i "UFW BLOCK"
```
```
... IN=virbr0 ... DPT=67 ...
```
`IN=` (not `OUT=`) plus a DHCP/DNS destination port confirms the block happened on the `input` hook. Reply traffic itself wasn't independently at risk — once the initial request is accepted, connection tracking classifies the reply as `established`/`related`, already accepted by both rulesets. The failure was entirely the *initial* inbound request being rejected before any reply was generated.

---

## 12. Resolution

Since UFW's own chain — not my `inet filter` table — was dropping the traffic, I fixed it inside UFW:

```bash
sudo ufw allow in on virbr+ to any port 67 proto udp comment 'libvirt dhcp'
sudo ufw allow in on virbr+ to any port 53 comment 'libvirt dns'
sudo ufw reload
```

The trailing `+` is UFW's interface-name-prefix wildcard, covering `virbr0`, `virbr1`, `virbr2`, and any bridge libvirt creates in the future, without needing to revisit these rules as new networks appear.

**Verification:**
```bash
sudo ufw show added
sudo journalctl -k -f
```
After restarting a VM (or forcing a DHCP renewal inside one), I look for no further `[UFW BLOCK]` entries on port 53/67, and confirm the VM gets a lease and working internet access.

**Caveat:** these rules match on interface-name prefix specifically, not on "a libvirt-managed bridge" in general — any virtual network using a bridge name that doesn't match `virbr*` needs its own corresponding UFW rule.

---

## 13. Consolidated architecture, including UFW

After Section 11's fix, two independent rule sets each separately have to permit VM traffic at every relevant hook:

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

*Note: the routed policy shown above (`allow`) is the permissive baseline that resolved Failure 2. Section 13 documents a subsequent hardening step replacing this global allow with a scoped, `virbr+`-only exception — later superseded in turn by Section 3's default-accept model.*

---

## 14. Hardening (superseded): replacing UFW's global routed policy with a scoped exception

Sections 9–12 left UFW's default forward policy at `allow (routed)` — sufficient to fix Failure 2, but broader than libvirt actually needs, since it permits forwarding between *any* pair of interfaces for *any* purpose.

Naively tightening this to `deny` also blocks the `virbr*` traffic VMs need. In practice this produced an asymmetric symptom: `ping 1.1.1.1` kept working from inside a guest, while DNS resolution and larger TCP flows (loading a web page) timed out — that asymmetry pointed at the routed policy itself as the new cause.

**The targeted fix I applied at the time:**
```bash
# 1. Lock down the global forwarding policy
sudo ufw default deny routed

# 2. Allow selective routing only for libvirt's bridges, with a descriptive comment
sudo ufw route allow in on virbr+ comment 'libvirt internet routing'

# 3. Apply the changes
sudo ufw reload
```

Resulting config:
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

> **Status: superseded, kept for reference.** This is the same allowlist-by-interface-name model as Section 15.3, applied to UFW's routed policy. It has the same limitation: it silently blocks any other forwarding-capable app (Docker, Kubernetes, LXD, …) unless I add a matching `ufw route allow` rule for it too. Section 3 explains why I now prefer the default-accept model instead of extending this allowlist further.

---

## 15. Lessons learned

1. libvirt manages firewall rules exclusively at runtime, via netlink — it never touches `/etc/nftables.conf`. I always check the live ruleset, not just static config.
2. Rules referencing dynamically-created interfaces need wildcard matching (`virbr*` in nftables, `virbr+` in UFW) — hardcoded names silently fail for any interface created afterward.
3. Multiple independently-managed rule sets can be evaluated at the same netfilter hook — internal correctness in one doesn't guarantee traffic is permitted if another (UFW, firewalld, Docker, a hand-written table) also evaluates the same hook.
4. Diagnosis should follow where traffic is actually consumed, not the symptom's surface description — "can't reach the internet" doesn't say whether the failure is at `input` or `forward`.
5. Kernel logs (`IN=`/`OUT=` plus destination port) identify the responsible hook directly — I check these before making config changes.
6. A global `allow (routed)` default is broader than libvirt requires, but tightening it to a `virbr*`-only allowlist (Section 13) doesn't scale to a host running an unknown/growing set of forwarding-capable apps — Section 3's default-accept model is what I use now.

---

## 16. Failure 1: hardcoded bridge interfaces

**The problem:** an early version of my `forward` chain enumerated bridges explicitly:
```nft
chain forward {
  policy drop;

  iifname virbr0 accept
  iifname virbr1 accept
}
```
At the time, only `virbr0` and `virbr1` existed, so this appeared to work.

**Why it breaks:** libvirt creates bridges dynamically as I define/start new virtual networks (`virbr2`, `virbr3`, …). A ruleset hardcoding specific names can't account for interfaces that don't exist yet when it's written — any new network created afterward has its traffic silently dropped by the `drop` default, despite libvirt's own NAT/routing being entirely correct.

**The fix — wildcard interface matching:**
```nft
iifname "virbr*" accept
oifname "virbr*" accept
```
This matches `virbr0`, `virbr1`, `virbr2`, and any future bridge, without needing the ruleset edited again.

**General principle:** when writing firewall rules for a subsystem that manages its own dynamic set of interfaces (libvirt, Docker, etc.), match on the interface *naming convention*, not specific instances.

> **⚠️ Warning: this chain's `policy drop` can silently break other kernel-forwarding applications.**
> The `chain forward { policy drop; ... }` block above is a separate, independently-evaluated ruleset (Section 1/12) sitting at the same `forward` hook as every other app on the host that routes or bridges traffic through the kernel. Because its default policy is `drop` and it only allowlists `virbr*`, any other app whose traffic hits the `forward` hook and doesn't match `virbr*` gets silently dropped here — even with entirely correct rules of its own.
>
> This happened to me in practice: this rule was written to fix libvirt DHCP/DNS at a time when Docker wasn't in use, and later broke Docker container networking entirely once Docker was installed, since `docker0` and Docker's per-network bridges (`br-xxxxxxxxxxxx`) don't match `virbr*`.
>
> **Other apps/scenarios using the `forward` hook that could break the same way:** Docker/Podman in rootful bridge mode (rootless Podman usually routes through `slirp4netns`/`pasta` instead, so it's unaffected); Kubernetes/k3s/minikube/kind (CNI bridges — `flannel`, `cilium`, `calico`, `cni0`); LXC/LXD (`lxcbr0`, `lxdbr0`); VirtualBox/VMware host-only or bridged networking (`vboxnet*`); WireGuard/OpenVPN configured to route for other hosts; manual IP forwarding/NAT-gateway setups; systemd-nspawn containers with bridged networking.
>
> **The general lesson:** a rule set scoped to "only allow the interfaces I currently care about" with a `drop` default is a *global* policy decision, not a scoped one. Before adding a `policy drop` forward chain like this, I audit what else on the host currently does (or might in the future do) kernel-level packet forwarding, and either widen the allowlist accordingly or prefer a less invasive default.

> **Status: superseded, kept for reference.** The wildcard fix above is still valid and still solves the immediate DHCP/DNS symptom, but the underlying default-drop-plus-allowlist *model* is no longer what I use, precisely because of the breakage described above — see Section 3 for the replacement approach. The wildcard-matching *technique* itself remains correct and is reused there.
