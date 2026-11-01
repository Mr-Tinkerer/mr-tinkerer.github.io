---
title: Virtual Machines Glossary
description: Terms used across the Virtual machines bucket.
tags:
  - glossary
---

## Overview

This page documents terms used across the Virtual machines bucket.

## Host CPU Passthrough

A hypervisor VM CPU-type setting that exposes the physical host CPU's exact instruction set/flags (e.g. AVX2) to the guest, instead of a generic/portable profile that only exposes a common baseline of instructions. This is used on the Local AI VM specifically because llama.cpp can use CPU instruction-set extensions the host actually has to speed up inference — a generic profile might hide those extensions from the guest and leave real performance on the table. The trade-off is that a VM configured this way can't be migrated to different physical hardware without matching (or superset) CPU features, since the guest is now depending on flags specific to this exact CPU. See the x86-64 microarchitecture level glossary entry for the portable alternative.
