---
title: Homelab Software Overview
description: Overview of all software systems in the home lab, including operating systems and services.
tags:
  - Homelab
  - Software
---

# Software Domains

This domain documents all software systems in the home lab, organized by type and function.

# Virtualization

## Proxmox

Primary hypervisor managing all VMs and containers in the home lab.

* [[Homelab/Software/Proxmox/Installing Proxmox on a Wi-Fi-Only Machine|Proxmox Installation]]

# Networking

## OpenWrt

Router operating system providing network infrastructure and Wi-Fi services.

* [[Homelab/Software/OpenWrt/OpenWrt Overview on Lenovo ThinkPad P52s|OpenWrt Overview]]
* [[Homelab/Software/OpenWrt/OpenWrt Glossary|OpenWrt Glossary]]

## Nginx

Reverse proxy and web server providing SSL certificates and service routing.

* [[Homelab/Software/Nginx|Nginx]]

## Tailscale

Mesh network and subnet routing for secure remote access.

* [[Homelab/Software/Tailscale|Tailscale]]

# Storage

## OpenMediaVault

Network-attached storage providing file sharing, backups, and media services.

* [[Homelab/Software/OpenMediaVault/OpenMediaVault-Glossary|OpenMediaVault Glossary]]
* [[Homelab/Software/OpenMediaVault/Openmediavault-installation on Dell Vostro 260s|OpenMediaVault Installation]]

# Containerization

Container runtime and management tools for service deployment.

* [[Homelab/Logical/Containerization/Tool-choices|Tool Choices: Podman vs Docker]]

# DNS and Networking

Network configuration, addressing plans, and DNS management.

* [[Homelab/Logical/Home-network/Dns-naming-scheme|DNS Naming Scheme]]
* [[Homelab/Logical/Home-network/Dhcp-plan|DHCP Plan]]
* [[Homelab/Logical/Home-network/Addressing-plan|Addressing Plan]]

# Virtual Machine Management

VM allocation, resource distribution, and virtualization configuration.

* [[Homelab/Logical/Virtual-machines/Vm-allocation|VM Allocation]]

# Service Documentation

## Nextcloud

Document management and file sharing service.

* [[Homelab/Platform/Productivity-VM/Nextcloud/Nextcloud-installation|Nextcloud Installation]]

## Stirling-PDF

PDF manipulation and conversion tool.

* [[Homelab/Platform/Productivity-VM/Stirling-pdf/Stirling-pdf-installation|Stirling-PDF Installation]]

## ConvertX

E-book conversion service.

* [[Homelab/Platform/Productivity-VM/Convertx/Convertx-installation|ConvertX Installation]]

## Llama.cpp

Local LLM inference engine.

* [[Homelab/Platform/Local-AI-VM/Llama.cpp|Llama.cpp]]

## Open-WebUI

LLM web interface.

* [[Homelab/Platform/Local-AI-VM/Open-webui|Open-WebUI]]

## SearXNG

Metasearch engine.

* [[Homelab/Platform/Local-AI-VM/Searxng|SearXNG]]

## Homepage

Personal homepage service.

* [[Homelab/Platform/Personal-VM/Homepage|Homepage]]

# Maintenance Notes

Software entries include installation procedures, configuration guides, troubleshooting, and known issues.
