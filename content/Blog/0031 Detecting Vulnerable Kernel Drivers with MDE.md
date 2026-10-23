---
title: "Detecting Vulnerable Kernel Drivers with MDE"
created: 2026-02-01
modified: 2026-02-01
tags: ["MDE", "SIEM"]
draft: false
---

## Introduction

I recently discovered that a-lot of threat actors are exploiting kernel vulnerabilities to escalate their privileges and to shutdown the EDR system running on endpoints. In this article I'll go through detecting vulnerable kernel drivers in our envioronment using Microsoft Defender for Endpoint.

## Technical Details

In Windows the kernel drivers have ring 0 privileges which is the highest privilege level. The threat actors are exploiting vulnerabilities within these kernels to elevate their privileges from a user-mode to kernel mode. This allows the threat actors to kill EDR systems and modify the EDR to not send data to the security provider. 

## Vulnerable Drivers

The [LOLDrivers](https://www.loldrivers.io/) has a complete list over all vulnerable kernel drivers and it comes with KQL query which allows us to detect vulnerable kernel drivers in our environment using `DeviceEvents` table.

![[0031 Detecting-Vulnerable-Kernel-Drivers-with-MDE.png]]

```sql
 let LOLDrivers = externaldata (Category:string, KnownVulnerableSamples:dynamic, Verified:string ) [h@"https://www.loldrivers.io/api/drivers.json"]
     with (
       format=multijson,
       ingestionMapping=@'[{"Column":"Category","Properties":{"Path":"$.Category"}},{"Column":"KnownVulnerableSamples","Properties":{"Path":"$.KnownVulnerableSamples"}},{"Column":"Verified","Properties":{"Path":"$.Verified"}}]'
     )
| mv-expand KnownVulnerableSamples
| extend SHA1 = tostring(KnownVulnerableSamples.SHA1), SHA256 = tostring(KnownVulnerableSamples.SHA256);

DeviceEvents
| where ActionType == "DriverLoad"
| join kind=inner (LOLDrivers | where isnotempty(SHA256)) on SHA256
| union (
  DeviceEvents
  | where ActionType == "DriverLoad"
  | join kind=inner (LOLDrivers | where isnotempty(SHA1)) on SHA1
)
| summarize arg_max(DeviceName, *) by ActionType, FileName, FolderPath, SHA256
| project DeviceName, ActionType, FileName, SHA256
```

The KQL query fetches known vulnerable kernel driver hashes from [LOLDrivers](https://www.loldrivers.io/) and then it compares the hashes to the ones that our endpoints recently loaded.

## Workbooks

I would highly recommend including the KQL inside a workbook as that will allow your colleagues to view all the devices whom have loaded a vulnerable devices into their system.

## Conclusion

Microsoft has implemented a blocklist for vulnerable kernel drivers but these blocklists aren't updated in real-time and sometimes these vulnerable kernel drivers are never added into the blocklist. However, using the KQL provided by LOLDrivers allows us to consistently monitor our environment for vulnerable kernel drivers and patch them quickly. 
