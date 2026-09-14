---
title: Homelab
description: Building an actual home lab — a repurposed laptop router, two donated PCs turned hypervisor and NAS, and the network/services layer tying them together.
tags:
  - Homelab
---
This domain covers how I stood up a home lab from scratch in a location with no wired internet: a laptop I turned into an [[Homelab/Software/OpenWrt/Openwrt-overview|OpenWrt]] router to bridge an enterprise Wi-Fi network onto a wired LAN, two free retired PCs I rebuilt into a [[Homelab/Software/Proxmox/Proxmox-installation|Proxmox]] hypervisor and an [[Homelab/Software/OpenMediaVault/Openmediavault-installation|OpenMediaVault]] NAS, the addressing/DNS/reverse-proxy/VPN mesh I layered on top, and the VMs and self-hosted services (file sync, document conversion, a dashboard, and locally-run AI models) running on that hypervisor.

The domain crosses the Hardware/Software/Logical threshold: four distinct physical machines, five bare-metal/router software entries, three active VM platforms, and three cross-cutting schemes.

# Hardware

Physical machines that make up the home lab, and the hardware-specific problems (not caused by any particular OS) hit while building them.

- [[Homelab/Hardware/Acer-Aspire-3/Acer-aspire-3-overview|Acer Aspire 3]] — the laptop originally meant to be the router; retired once it turned out to have no Ethernet port. ([[Homelab/Hardware/Acer-Aspire-3/Acer-aspire-3-glossary|Glossary]])
- [[Homelab/Hardware/Lenovo-Thinkpad-P52s/Lenovo-thinkpad-p52s-overview|Lenovo ThinkPad P52s]] — the laptop that actually became the router, including its Secure Boot and single-Wi-Fi-radio gotchas. ([[Homelab/Hardware/Lenovo-Thinkpad-P52s/Lenovo-thinkpad-p52s-glossary|Glossary]] · [[Homelab/Hardware/Lenovo-Thinkpad-P52s/Lenovo-thinkpad-p52s-gotchas|Gotchas]])
- [[Homelab/Hardware/Dell-Optiplex-5060/Dell-optiplex-5060-overview|Dell Optiplex 5060]] — the Proxmox hypervisor host. ([[Homelab/Hardware/Dell-Optiplex-5060/Dell-optiplex-5060-glossary|Glossary]] · [[Homelab/Hardware/Dell-Optiplex-5060/Build-and-repairs|Build and repairs]])
- [[Homelab/Hardware/Dell-Vostro-260s/Dell-vostro-260s-overview|Dell Vostro 260s]] — the NAS build, rebuilt across three cases, two power supplies, and two motherboards. ([[Homelab/Hardware/Dell-Vostro-260s/Dell-vostro-260s-glossary|Glossary]] · [[Homelab/Hardware/Dell-Vostro-260s/Build-and-repairs|Build and repairs]] · [[Homelab/Hardware/Dell-Vostro-260s/Dell-vostro-260s-gotchas|Gotchas]])

# Software

Software that runs directly on its own dedicated physical hardware rather than as a guest on the Proxmox hypervisor — the router's OS/services, and the hypervisor and NAS OSes themselves. See [[Homelab/Platform/Productivity-VM/Nextcloud/Nextcloud-installation|Platform]] below for everything that runs as a guest VM on Proxmox instead.

- [[Homelab/Software/OpenWrt/Openwrt-overview|OpenWrt]] — the router OS: installation, Wi-Fi driver bring-up, enterprise Wi-Fi authentication, GRUB, SSH, DHCP/DNS, and system maintenance. ([[Homelab/Software/OpenWrt/Openwrt-glossary|Glossary]])
- [[Homelab/Software/Nginx/Reverse-proxy-setup|Nginx]] — reverse proxy and HTTPS/private-CA setup for home lab services, running on the router. ([[Homelab/Software/Nginx/Nginx-glossary|Glossary]])
- [[Homelab/Software/Tailscale/Tailscale-setup|Tailscale]] — mesh VPN for reaching the home lab from outside the building, running on the router. ([[Homelab/Software/Tailscale/Tailscale-glossary|Glossary]])
- [[Homelab/Software/Proxmox/Proxmox-installation|Proxmox]] — the hypervisor itself, including VM creation. ([[Homelab/Software/Proxmox/Proxmox-glossary|Glossary]])
- [[Homelab/Software/OpenMediaVault/Openmediavault-installation|OpenMediaVault]] — the NAS OS: install, networking, the RAID/BTRFS storage pool, shares, SMART monitoring, and SSH hardening. ([[Homelab/Software/OpenMediaVault/Openmediavault-glossary|Glossary]])

# Platform

Each entry is one VM running on the Proxmox hypervisor, bundling its guest OS's own bucket with one bucket per service it hosts — see [[Homelab/Logical/Containerization/Tool-choices|the Platform-buckets convention]] for why this domain groups VM-hosted software this way instead of one flat list. The Monitoring VM and Arrrrgh VM aren't listed here yet — see [[Homelab/Logical/Virtual-machines/Vm-allocation|Virtual machines]] for their roster entries; each has only a guest OS installed so far, with no services configured.

- **Productivity VM** ([[Homelab/Platform/Productivity-VM/Debian/Debian-overview|Debian]]) — [[Homelab/Platform/Productivity-VM/Nextcloud/Nextcloud-installation|Nextcloud]] ([[Homelab/Platform/Productivity-VM/Nextcloud/Nextcloud-glossary|Glossary]] · [[Homelab/Platform/Productivity-VM/Nextcloud/Nextcloud-gotchas|Gotchas]]), [[Homelab/Platform/Productivity-VM/Stirling-pdf/Stirling-pdf-installation|Stirling PDF]] ([[Homelab/Platform/Productivity-VM/Stirling-pdf/Stirling-pdf-glossary|Glossary]]), and [[Homelab/Platform/Productivity-VM/Convertx/Convertx-installation|ConvertX]] ([[Homelab/Platform/Productivity-VM/Convertx/Convertx-glossary|Glossary]]).
- **Personal VM** ([[Homelab/Platform/Personal-VM/Opensuse/Opensuse-gotchas|openSUSE]]) — [[Homelab/Platform/Personal-VM/Homepage/Homepage-setup|Homepage]], the dashboard linking every home lab service together. ([[Homelab/Platform/Personal-VM/Homepage/Homepage-glossary|Glossary]])
- **Local AI VM** ([[Homelab/Platform/Local-AI-VM/Fedora/Cpu-configuration|Fedora]]) — [[Homelab/Platform/Local-AI-VM/Llama.cpp/Build-and-cpu-tuning|Llama.cpp]] for CPU-only local LLM inference ([[Homelab/Platform/Local-AI-VM/Llama.cpp/Llama-cpp-glossary|Glossary]]), [[Homelab/Platform/Local-AI-VM/Open-webui/Open-webui-setup|Open WebUI]] as its chat front-end ([[Homelab/Platform/Local-AI-VM/Open-webui/Open-webui-glossary|Glossary]]), and [[Homelab/Platform/Local-AI-VM/Searxng/Searxng-setup|SearXNG]] as its private search backend ([[Homelab/Platform/Local-AI-VM/Searxng/Searxng-glossary|Glossary]]).

# Logical

Cross-cutting schemes that span multiple hardware and software entries above.

- [[Homelab/Logical/Home-network/Addressing-plan|Home-network]] — the subnet/addressing plan, DHCP range plan, and custom DNS naming scheme shared by every device in the home lab. ([[Homelab/Logical/Home-network/Home-network-glossary|Glossary]])
- [[Homelab/Logical/Virtual-machines/Vm-allocation|Virtual machines]] — the roster of VMs running on Proxmox, their roles, and their guest OSes. ([[Homelab/Logical/Virtual-machines/Virtual-machines-glossary|Glossary]])
- [[Homelab/Logical/Containerization/Tool-choices|Tool-choice philosophy]] — why each VM runs a different Linux distro and why every containerized service runs on Podman/Quadlet instead of Docker: the point was to deliberately learn unfamiliar tools, not chase technical superiority. ([[Homelab/Logical/Containerization/Containerization-glossary|Glossary]])
