---
title: "Notepad++ Security Incident Threat Hunting using KQL"
created: 2026-02-03
modified: 2026-02-03
tags: ["SIEM", "KQL"]
draft: false
---

## Introduction

Notepad++ infrastructure was compromised by state sponsored hackers where they redirected the update traffic from notepad-plus-plus.org to an site which downloads `update.exe` executable program. According to [Rapid7 Labs](https://www.rapid7.com/blog/post/tr-chrysalis-backdoor-dive-into-lotus-blossoms-toolkit/) the executable program which is responsible for updating Notepad++ (`gup.exe`) executed a suspicious process `update.exe` which was downloaded from `95.179.213.0`.

## Threat Hunting

Using Azure Sentinel from Microsoft Security Portal, we can craft custom queries to investigate if any of our endpoints were affected by the Notepad++ security incident. However, we first need to identify all our endpoints that has Notepad++ installed on them.

```sql title="DeviceTvmSoftwareInventory"
DeviceTvmSoftwareInventory
| where SoftwareName contains "notepad"
| where not(SoftwareVersion == "8.9.1")
| project DeviceId, DeviceName, OSVersion, SoftwareVendor, SoftwareName, SoftwareVersion
| summarize total = count() by SoftwareVersion
```

We can now investigate the process `gup.exe` to see if it made any connections to unauthorized URLs which is possible with the following query.

```sql title="DeviceNetworkEvents"
let AllowedUrls = dynamic(["notepad-plus-plus.org", "github.com", "release-assets.githubusercontent.com", "raw.githubusercontent.com"]);
DeviceNetworkEvents
| where TimeGenerated > ago(360d)
| where InitiatingProcessCommandLine startswith '"gup.exe"'
| where not(RemoteUrl has_any (AllowedUrls))
| where RemoteIPType <> "Loopback"
| where isnotempty(RemoteIP)
```

We can also use the following query to identify all the executables programs, dynamic link libraries, and other files the process `gup.exe` made in our endpoints. 

```sql title="DeviceFileEvents"
DeviceFileEvents
| where TimeGenerated > ago(360d)
| where InitiatingProcessFileName startswith "gup.exe"
```

The [Rapid7 Labs](https://www.rapid7.com/blog/post/tr-chrysalis-backdoor-dive-into-lotus-blossoms-toolkit/) and [SecureList](https://securelist.com/notepad-supply-chain-attack/118708/) researched the Notepad++ Security Incident and provided some IoCs which we can use to investigate if any of our endpoints were affected.

```sql title="DeviceNetworkEvents - IPv4"
let ioc = dynamic(["124.222.137.114", "59.110.7.32", "61.4.102.97", "95.179.213.0", "45.76.155.202", "45.76.155.202",  "95.179.213.0", "45.77.31.210"]);
DeviceNetworkEvents
| where TimeGenerated > ago(360d)
| where RemoteIP has_any (ioc)
```

```sql title="DeviceNetworkEvents - URLs"
let ioc = dynamic(["api.skycloudcenter.com", "api.wiresguard.com", "cdncheck.it.com", "safe-dns.it.com"]);
DeviceNetworkEvents
| where TimeGenerated > ago(360d)
| where RemoteUrl has_any (ioc)
```

```sql title="DeviceFileEvents"
let ioc = dynamic(["a511be5164dc1122fb5a7daa3eef9467e43d8458425b15a640235796006590c9", 
"8ea8b83645fba6e23d48075a0d3fc73ad2ba515b4536710cda4f1f232718f53e", 
"2da00de67720f5f13b17e9d985fe70f10f153da60c9ab1086fe58f069a156924", 
"77bfea78def679aa1117f569a35e8fd1542df21f7e00e27f192c907e61d63a2e", 
"3bdc4c0637591533f1d4198a72a33426c01f69bd2e15ceee547866f65e26b7ad", 
"9276594e73cda1c69b7d265b3f08dc8fa84bf2d6599086b9acc0bb3745146600", 
"f4d829739f2d6ba7e3ede83dad428a0ced1a703ec582fc73a4eee3df3704629a", 
"4a52570eeaf9d27722377865df312e295a7a23c3b6eb991944c2ecd707cc9906", 
"831e1ea13a1bd405f5bda2b9d8f2265f7b1db6c668dd2165ccc8a9c4c15ea7dd", 
"0a9b8df968df41920b6ff07785cbfebe8bda29e6b512c94a3b2a83d10014d2fd", 
"4c2ea8193f4a5db63b897a2d3ce127cc5d89687f380b97a1d91e0c8db542e4f8", 
"e7cd605568c38bd6e0aba31045e1633205d0598c607a855e2e1bca4cca1c6eda", 
"078a9e5c6c787e5532a7e728720cbafee9021bfec4a30e3c2be110748d7c43c5", 
"b4169a831292e245ebdffedd5820584d73b129411546e7d3eccf4663d5fc5be3", 
"7add554a98d3a99b319f2127688356c1283ed073a084805f14e33b4f6a6126fd", 
"fcc2765305bcd213b7558025b2039df2265c3e0b6401e4833123c461df2de51a",
"C7CC87EF3829A33B7F178D88A71BA548C37020005B09D16A76FCD356621335E6",
"51266007C039AB80DBE9A2C38ED75759D954458D8864A0429C71E87BE2BDDCE2",
"69CAA18EC5E86CF3A7376F3A9A08D118CBADE608432DC262BA6C7FE692DA7D33",
"A3CF1C86731703043B3614E085B9C8C224D4125370F420AD031AD63C14D6C3EC",
"798FD7C2A2D4F0865AEC808962489B39F995961E38E2BEBDA8F84DDC5A935D86",
"4D4AEC6120290E21778C1B14C94AA6EBFF3B0816FB6798495DC2EAE165DB4566"]);
DeviceFileEvents
| where TimeGenerated > ago(360d)
| where SHA256 has_any (ioc)
```

If no endpoints in your organizations made connections to these URLs or IPv4 addresses, and the file hash doesn't exist in your environment, it's safe to assume your endpoints were not affected by the incident.

## Mitigation

I would recommend updating the Notepad++ application profile on Microsoft Intune which the latest version of Notepad++ as it comes with improved security controls for updates. 

## Conclusion

Notepad++ is an application millions of people uses and unfortunately their infrastructure was compromised which lead to some endpoints being compromised. All we can do now is to investigate our environment and see if any endpoints where affected by the security incident.