---
title: Windows Server 2019 Installation and Configuration on Arrrrgh VM
description: Configuration guide for the Windows Server guest OS running on the Arrrrgh VM.
tags:
  - Homelab
  - Platform
  - Arrrrgh-VM
  - Windows-Server
  - Guest-OS
---

## Overview

The Windows Server guest OS hosts the complete ARR (App Repository & Remote) stack, providing the foundation for automated media management and organization on the Arrrrgh VM.

## Security Configuration

### Disable Windows Defender

Windows Defender real-time protection interferes with the ARR stack executables and causes false positive detections.

**Implementation Protocol:**

1. **Access Security Settings:** Launch Windows Security from the Start menu search.
2. **Navigate to Virus & Threat Protection:** Select the main protection dashboard.
3. **Access Management Options:** Click "Manage settings" under Virus & threat protection.
4. **Disable Real-Time Protection:** Toggle off "Real-time protection".
5. **Disable Additional Scans:** Disable Cloud Delivered Protection, Controlled Folder Access, and Core Isolation.

> [!warning] **Caveat: Security Implications**
> Disabling Windows Defender reduces system security for a trusted homelab environment. This configuration assumes physical or network isolation from external threats. Re-enable protection before connecting the VM to untrusted networks.

### Firewall Configuration

The Windows Firewall blocks outbound connections from ARR stack services, preventing web interface access and API communication.

**Implementation Protocol:**

1. **Open Advanced Security:** Launch Windows Defender Firewall with Advanced Security from Start menu search.
2. **Create Inbound Rule:** Navigate to Inbound rules and select "New Rule..."
3. **Select Program Type:** Choose "Program" as the rule type.
4. **Specify Program Path:** Browse to the executable path:
   - `C:\ProgramData\Prowlarr\bin\Prowlarr.Console.exe`
   - `C:\ProgramData\Deluge\deluge-web.exe`
   - `C:\ProgramData\Sonarr\bin\Sonarr.Console.exe`
   - `C:\ProgramData\Radarr\bin\Radarr.Console.exe`
   - `C:\ProgramData\Bazarr\bin\Bazarr.Console.exe`
5. **Allow Connection:** Select "Allow the connection" and proceed.
6. **Profile Selection:** Apply to Domain, Private, and Public profiles.
7. **Name and Description:** Enter a descriptive name such as `Prowlarr Web UI Access`.
8. **Repeat:** Create separate rules for each ARR stack service.

## NFS Client Configuration

The Windows NFS client requires explicit configuration to mount remote shares with anonymous user credentials matching the host's nobody group (UID 1000).

### Install NFS Client Role

1. **Open Server Manager:** Launch Server Manager from the Start menu.
2. **Add Roles and Features:** Navigate to Manage → Add roles and features.
3. **Install Features:** Proceed through the wizard until the Features page.
4. **Select Client for NFS:** Check "Client for NFS" and complete installation.
5. **Verify Installation:** Confirm the NFS client service is running in Services.msc.

### Configure Anonymous User Mapping

1. **Open Registry Editor:** Launch regedit.exe with administrative privileges.
2. **Navigate to NFS Registry Path:** Access `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default`.
3. **Create AnonymousUid Value:** Create a new DWORD (32-bit) value named `AnonymousUid` with decimal value `1000`.
4. **Create AnonymousGid Value:** Create a new DWORD (32-bit) value named `AnonymousGid` with decimal value `1000`.
5. **Restart NFS Service:** Execute `nfsadmin client restart` in PowerShell to apply changes.

### Mount Remote NFS Share

Execute the following command in PowerShell with administrative privileges:

```powershell
mount.exe -o anon \\10.10.0.13\export\Arrrrgh Z:
```

**Unmount Command:**
```powershell
umount Z:
```

## Task Scheduler Configuration

Windows Task Scheduler automates NFS mount operations at boot time, ensuring network shares are available before ARR stack services start.

### Create Boot-Time NFS Mount Task

1. **Open Task Scheduler:** Launch Task Scheduler from Start menu search.
2. **Create New Task:** Select "Create Task" (not "Create Triggered Task").
3. **Configure General Settings:**
   - Name: `Mount NFS Arrrrgh share on boot`
   - Select "Run with highest privileges"
   - Select "Run whether user is logged on or not"
4. **Configure Trigger:**
   - Navigate to the Triggers tab
   - Select "New"
   - Set "Begin the task" to "At startup"
5. **Configure Action:**
   - Navigate to the Actions tab
   - Select "New"
   - Program/Script: `powershell.exe`
   - Add arguments: `-Command "mount.exe -o anon \\10.10.0.13\export\Arrrrgh Z:"`
6. **Finalize:** Click OK and enter administrative credentials when prompted.
7. **Verify Permissions:** Confirm the `nobody` group on the NFS share has read and execute permissions.
8. **Test Configuration:** Reboot the VM and verify the Z: drive mounts automatically.

### Disable Server Manager Auto-Start

The Server Manager process consumes CPU resources during idle periods and interferes with other administrative tasks.

**Implementation Protocol:**

1. **Access Task Scheduler:** Launch Task Scheduler with administrative privileges.
2. **Navigate to Server Manager Library:** Browse to Task Scheduler Library → Microsoft → Windows → Server Manager.
3. **Disable Task:** Right-click the `ServerManager` task and select "Disable".
4. **Verify:** Confirm the task status shows "Disabled" in the Tasks column.

## Service Integration

The ARR stack services integrate through API connections managed by Prowlarr:

1. **Prowlarr** coordinates indexer settings across all services
2. **ARR services** (Sonarr, Radarr, Bazarr) connect to Prowlarr to sync indexer configurations
3. **Deluge** handles actual torrent and Usenet downloads
4. **Sonarr/Radarr** manage download organization and metadata retrieval
5. **Bazarr** retrieves subtitles based on Sonarr/Radarr library needs
