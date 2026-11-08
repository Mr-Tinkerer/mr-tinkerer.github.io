---
title: Mullvad VPN Setup on Arrrrgh VM
description: Configuration guide for Mullvad VPN on the Arrrrgh VM, including auto-connect on boot.
tags:
  - Arrrrgh-VM
  - Mullvad
  - VPN
---

## Overview

This guide documents the setup of Mullvad VPN on the Arrrrgh VM, including enabling local network sharing, lockdown mode, and automatic connection on system boot.

---

## Prerequisites

* Access to the Arrrrgh VM via RDP or NoVNC
* Mullvad VPN Windows application installed on the VM

---

## Initial Configuration

### Install and Launch

1. Download the Mullvad Windows application from the official source
2. Launch the application
3. Log in with your Mullvad account credentials
4. Test the VPN connection to verify functionality

---

## Network Connectivity Fix

### Observed Issue

Initial testing revealed that network connections dropped when disconnecting through the Remmina RDP client.

**Error State:** Connection drops unexpectedly during RDP session termination.

**Isolation:** The issue occurs specifically when using RDP for remote access.

### Solution

Enable the following settings within the Mullvad application:

1. Navigate to **Settings → VPN Settings**
2. Enable **Local network sharing**
3. Retest the connection

**Result:** Network connectivity is maintained when disconnecting through RDP.

---

## Daemon Verification

A Mullvad daemon process runs in Task Manager when the service is active.

---

## Auto-Connect Configuration

### Windows Task Scheduler Setup

Configure the system to automatically connect to Mullvad VPN at boot:

1. **Open Task Scheduler:** Launch from Start menu search
2. **Create New Task:** Select "Create Task" (not "Create Triggered Task")

#### General Settings

* Name: `Connect to Mullvad VPN on boot`
* Select "Run with highest privileges"
* Select "Run whether user is logged on or not"

#### Trigger Configuration

1. Navigate to the Triggers tab
2. Select "New"
3. Set "Begin the task" to "At startup"
4. Configure delay: 1 minute after boot

#### Action Configuration

1. Navigate to the Actions tab
2. Select "New"
3. **Program/Script:** `powershell.exe`
4. **Add arguments:** `-Command "mullvad connect"`

#### Finalization

Click OK and enter administrative credentials when prompted.

---

## Verification

Confirm successful setup:

1. Reboot the Arrrrgh VM
2. Verify Mullvad connects automatically within 1 minute
3. Test network connectivity
