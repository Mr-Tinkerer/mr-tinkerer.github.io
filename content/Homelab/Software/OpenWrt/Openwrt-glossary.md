---
title: OpenWrt Glossary
description: Terms used across the OpenWrt bucket.
tags:
  - glossary
---
## OpenWrt

A Linux-based OS designed for embedded/router hardware but also installable on x86 machines; built to run in very limited storage/RAM compared to a general-purpose distro. See [OpenWrt (Wikipedia)](https://en.wikipedia.org/wiki/OpenWrt).

## pfSense

An open-source FreeBSD-based router/firewall OS, one of the oldest in the DIY-router space. Requires an internet connection during install, which ruled it out for this home lab (no wired connection available at install time). See [pfSense (Wikipedia)](https://en.wikipedia.org/wiki/PfSense).

## OPNsense

A 2015 fork of pfSense, created over concerns about pfSense's build-tooling transparency and licensing changes. Its FreeBSD base has inconsistent Wi-Fi driver support, which ruled it out here. See [OPNsense (Wikipedia)](https://en.wikipedia.org/wiki/OPNsense) and [the OPNsense fork history](https://docs.opnsense.org/history/thefork.html).

## SquashFS

A read-only, compressed filesystem commonly used for embedded firmware images. Not chosen for this router because the Wi-Fi driver packages needed to be installed after the fact, which requires a writable root filesystem. See [SquashFS (Wikipedia)](https://en.wikipedia.org/wiki/SquashFS).

## UCI

OpenWrt's Unified Configuration Interface — the `uci` command-line tool and underlying config file format used to read and change nearly all OpenWrt settings from the shell. See [UCI (OpenWrt docs)](https://openwrt.org/docs/guide-user/base-system/uci).

## LuCI

OpenWrt's web-based administration UI — the `https://<router-ip>` login page I use for most day-to-day router configuration (interfaces, DHCP/DNS, Wi-Fi, package installs) instead of the shell. It's convenient, but it isn't a thin wrapper over `uci`: it has its own config-generation logic, which is what caused the [[Homelab/Software/OpenWrt/Enterprise-wifi-authentication|enterprise Wi-Fi authentication bug]] — LuCI wrote an invalid value to the underlying config that I had to correct by hand with `uci` instead.

## apk

The package manager used by modern OpenWrt releases, used throughout this bucket to install things the base image doesn't ship (Wi-Fi drivers/firmware, `wpa_supplicant`, `coreutils-base64`, etc.). Normally it fetches packages and verifies signatures against a trusted key over the network; when I installed [[Homelab/Software/OpenWrt/Wifi-driver-installation|Wi-Fi driver packages]] before the router had working internet, I had to download the `.apk` files on another machine, copy them over, and install with `--allow-untrusted` since manually-downloaded packages aren't signed by a key `apk` recognizes in that offline flow. See [apk (OpenWrt docs)](https://openwrt.org/docs/guide-user/additional-software/apk).
