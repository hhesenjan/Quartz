---
title: "Troubleshooting Microsoft Defender for Identity Sensor v3.x RPC Audit Misconfigured"
created: 2026-08-14
modified: 2026-08-14
tags: ["MDE", "Identity"]
draft: false
---

## Introduction

Microsoft Defender for Identity is a product that comes with Microsoft Defender XDR. Its primary capabilities is to analyze your on-premise (domain server) to search for misconfigurations. While setting up the Microsoft Defender for Identity Sensor I was faced with a alert which said the following *"Identity Sensor v3.x RPC Audit Misconfigured"*. In this article I'll go through resolving the issue.

## What is RPC?

Remote Procedure Call (RPC) is a communication protocl which uses TCP or UDP to communicate with an program running in a different system. RPC uses the client-server model where the client communicates with the server for the services. As an example the client invokes a function and the server executes the function and returns the data to the client.

Microsoft uses RPC protocol for many of its components such as AD Replication, Print Spooler, Account and Policy Lookup and so forth. This is why threat actors are interested in targeting services running RPC and that is why Microsoft wants to monitor it.

## Solution

This section of the document goes through resolving the alert.

1. Go to [Microsoft Security](https://security.microsoft.com/machines).
    ![[0052 Microsoft Defender for Identity Sensor v3.x RPC Audit Misconfigured 01.png]]

2. Click on **Assets -> Devices**.
    ![[0052 Microsoft Defender for Identity Sensor v3.x RPC Audit Misconfigured 02.png]]

3. Find your domain controller.
    ![[0052 Microsoft Defender for Identity Sensor v3.x RPC Audit Misconfigured 03.png]]

4. Click on **"..." -> Manage Tags**.
    ![[0052 Microsoft Defender for Identity Sensor v3.x RPC Audit Misconfigured 04.png]]

5. Add **"Unified Sensor RPC Audit"** tag.
    ![[0052 Microsoft Defender for Identity Sensor v3.x RPC Audit Misconfigured 05.png]]

The tag synchronization should take around 15 minutes and from there on you can close the alert from Microsoft. If an new alert is generated make sure that there are no whitepsaces on right or left side of the tag.

## Conclusion

Microsoft recommends us to add the **Unified Sensor RPC Audit** tag since that will allow the identity sensor to monitor RPC traffics. It's crucial for us to monitor the RPC traffics happening in our environment since many of the Windows and Active Directory components uses it. Additionally, the upcomming newer identity sensor will start monitoring RPC traffics by default without the tag. Hopefully this article was useful for you in some way!

## See also

- [[0018 Onboarding Microsoft Defender for Identity v2.x Sensor.md|Onboarding Microsoft Defender for Identity v2.x Sensor]]
- [[0055 Deep Dive Into Microsoft Defender for Identity Capabilities.md|Deep Dive Into Microsoft Defender for Identity Capabilities]]