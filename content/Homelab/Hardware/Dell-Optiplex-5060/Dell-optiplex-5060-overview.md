---
title: Dell Optiplex 5060 Overview
description: A free retired office PC that became the home lab's Proxmox hypervisor.
tags:
  - Homelab
  - Hardware
---

## Overview

This page documents a Dell Optiplex 5060, a free retired office PC that became the home lab's Proxmox hypervisor.

## What It Is

A Dell Optiplex 5060 got free from a college IT department clearing out its retired-PC "graveyard." It has a proprietary (non-standard-form-factor) motherboard, so its board can't be moved into a different case — unlike the Dell Vostro 260s, which uses a normal ATX board. It supports DDR4 RAM and has a better CPU than the Vostro, so it was designated the Proxmox hypervisor for the home lab.

## Condition as Received

The PC arrived with a Seagate 500GB hard drive but no RAM sticks — the donor department's policy required removing any DDR4 RAM or SSD/NVMe storage before giving a machine away, due to a RAM shortage at the time. This meant the machine couldn't boot until replacement RAM was sourced.

See [missing RAM screenshot](Images/optiplex-missing-ram-sticks-as-received.png) for the Optiplex as received — no RAM installed, per the donor's take-back policy.

Its CPU is an Intel Core i7-8700, later relevant to the Local AI VM's CPU-only inference performance since that VM runs on this host.

See the Build and repairs page for how it was brought up to a working state.
