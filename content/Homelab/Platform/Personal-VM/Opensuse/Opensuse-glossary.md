---
title: openSUSE Glossary
description: Terms specific to the openSUSE guest OS running on the Personal VM.
tags:
  - Homelab
  - Opensuse
---
## firewalld

The dynamic firewall management daemon that openSUSE (like Fedora/RHEL-family distros) ships enabled by default. Unlike a bare `iptables`/`nftables` ruleset, it manages rules through "zones" and services rather than raw rules, and it's active out of the box — meaning any port a freshly installed service needs has to be explicitly opened (or the daemon disabled) before that service is reachable from the network. See the [firewalld documentation](https://firewalld.org/documentation/).

## SELinux

Security-Enhanced Linux, a mandatory access control (MAC) kernel module that enforces fine-grained policy on top of normal Unix permissions — even a process running as root can be blocked from doing something a plain permissions check would allow. openSUSE ships it enabled and in **enforcing** mode by default, which matters here because a container's mounted volume can be fully accessible by normal Unix permissions and still get blocked from being written to, since SELinux's policy hasn't been told the container process is allowed to touch that file context. See [Understanding SELinux (Red Hat docs)](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/using_selinux/getting-started-with-selinux_using-selinux).

## `:Z` volume relabel suffix

A Podman/Docker volume-mount flag that tells the container runtime to relabel the mounted host directory with an SELinux context the container is permitted to write to. It's the standard fix for a Podman quadlet or `podman run` command failing to write into a bind-mounted directory on an SELinux-enforcing host — see [[Homelab/Platform/Personal-VM/Opensuse/Opensuse-gotchas|Gotchas]] for where I hit this.
