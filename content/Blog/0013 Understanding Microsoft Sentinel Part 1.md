---
title: "Understanding Microsoft Sentinel KQL"
created: 2025-09-24
modified: 2025-09-24
tags: ["SIEM", "KQL"]
draft: false
---

## Introduction

Currently, in the organization we are forwarding all data from Windows, Cortex XDR, and Office 365 into Microsoft Sentinel so that our third-party SOC analysts can warn us about malicious and suspicious behaviours. In this article I'll go through some Kusto Query Language (KQL) which can be useful during an incident.

## Kusto Query Language

Kusto Query Language (KQL) in Microsoft Sentinel enables us to filter thousands to millions of log data. A strong understanding of KQL will enable us to quickly investigate an incident once it occurs.

### Windows Security Events

To monitor malicious account modifications and password changes.

```sql title="Windows Security Events"
SecurityEvent 
| where TargetAccount in ("COSMOS\\husenjan", "COSMOS\\jdoe")  and EventID in (4738, 4724)
| where TimeGenerated between (datetime("2025-09-12") .. datetime("2025-09-16"))
```

### Office 365 Sign-Ins

To monitor a user's sign-ins occurring in the Office 365 environment.

```sql title="Office 365 Sign-In"
SigninLogs 
| where UserPrincipalName == "husenjan@cosmos.local" 
| where TimeGenerated between (datetime("2025-09-12") .. datetime("2025-09-16"))
```

### Phishing Emails

To investigate phishing emails.

```sql title="Phishing Emails"
EmailEvents 
| project TimeGenerated, AuthenticationDetails, DeliveryAction, DeliveryLocation, OrgLevelAction, OrgLevelPolicy, RecipientEmailAddress, SenderFromAddress, SenderIPv4, SenderMailFromDomain, To, Cc 
| where SenderFromAddress == "malicious-email@evil-website.local" 
```

```sql title="Phishing Emails"
EmailEvents 
| project TimeGenerated, AuthenticationDetails, DeliveryAction, DeliveryLocation, OrgLevelAction, OrgLevelPolicy, RecipientEmailAddress, SenderFromAddress, SenderIPv4, SenderMailFromDomain, To, Cc 
| where SenderIPv4 == "10.10.19.4" 
```

### URL Clicks

To investigate which users clicked on the malicious URLs sent to their emails.

```sql title="URL Clicks"
URLClickEvents
| where Url has "evil-site.com"
```

### Data Exfilitration

To investigate data exfilitrations occurring in the organization.

```sql title="Data Exfilitration"
DeviceFileEvents 
| project ActionType, DeviceName, InitiatingProcessAccountName, FileName, FolderPath, InitiatingProcessVersionInfoOriginalFileName 
| where FolderPath has "D:\\"
```

## Conclusion

With the right hads Microsoft Sentinel is a extremely powerfull tool that enables us to search through thousands of logs within seconds. A stong understanding about KQL enables us to quickly investigate the affected users, understand the incident, and remediate the incident. 

If your organization is considering adding Microsoft Sentinel - I highly recommend them adding it because it enables us to search through thousands of logs within seconds which saves a-lot of time.