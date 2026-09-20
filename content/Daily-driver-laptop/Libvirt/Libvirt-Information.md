---
title: Libvirt Information
description: Important information relating to libvirt
tags:
  - Virtual_Machines
  - libvirt
  - information
---
## libvirt network modes
| Libvirt Networking Mode | Description                                                     | Best For                         |
| ----------------------- | --------------------------------------------------------------- | -------------------------------- |
| **NAT**                 | Simple, default-style networking with outbound internet access  | General-purpose VMs              |
| **Routed**              | VMs use routed IP addresses without traditional NAT             | Servers and routable VM networks |
| **Open**                | Host bridge with minimal libvirt filtering                      | Advanced/custom networking       |
| **Isolated**            | VMs can communicate with each other but have no external access | Testing and private networks     |
| **SR-IOV**              | Direct NIC virtual functions for near-native performance        | High-performance workloads       |


# VM Firmware
|Firmware option|Firmware type|Flash size|Secure Boot|Typical use|
|---|---|---|---|---|
|**BIOS**|Legacy PC BIOS|—|❌ No|Older OSes / maximum legacy compatibility|
|**UEFI**|OVMF UEFI|2 MiB|❌ No|Legacy UEFI configuration / compatibility|
|**`OVMF_CODE.4m.fd`**|OVMF UEFI|**4 MiB**|❌|Modern UEFI without Secure Boot|
|**`OVMF_CODE.secboot4m.fd`**|OVMF UEFI|**4 MiB**|✅ Yes|Modern UEFI + Secure Boot|

# Chipset
|Chipset|QEMU machine type|Description|PCIe|Typical use|
|---|---|---|---|---|
|**i440FX**|`pc-i440fx-*`|Older, traditional PC chipset emulation|❌|Legacy compatibility|
|**Q35**|`pc-q35-*`|Newer Intel Q35 chipset model|✅|Modern VMs

# Libvirt Snapshots
[[Libvirt-external-snapshots]]
