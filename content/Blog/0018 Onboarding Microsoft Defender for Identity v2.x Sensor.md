---
title: "Onboarding Microsoft Defender for Identity v2.x Sensor"
created: 2025-10-22
modified: 2025-10-22
tags: ["MDE", "IDENTITY"]
draft: false
---

## Introduction

Microsoft Defender for Identity comes with the Microsoft Defender XDR suite. It uses the signals from Windows Active Directory to detect, investigate, and respond to cyberthreats.

It will help with detecting, investigating, and responding to attacks such as reconnaissance, lateral movements, brute-force attacks. Additionally, it will be able to use signals from Azure Active Directory, Windows Active Directory, and endpoints to reference affected users, devices, and services.

## Onboarding

1. Go to **Settings -> Identities**.
	![[0018 Onboarding-Microsoft-Defender-for-Identity-01.png]]

2. Go to Sensors -> Add Sensor and Download Azure ATP Sensor.
	![[0018 Onboarding-Microsoft-Defender-for-Identity-02.png]]

3. Transfer `Azure ATP Sensor.zip` file to Domain Controller and run `Azure ATP Sensor Setup.msi` as administrator.
	![[0018 Onboarding-Microsoft-Defender-for-Identity-03.png]]

4. Click on **Yes**.
	![[0018 Onboarding-Microsoft-Defender-for-Identity-04.png]]

5. Select **English** and click on **Next**.
	![[0018 Onboarding-Microsoft-Defender-for-Identity-05.png]]

6. Click on **Next**. 
	![[0018 Onboarding-Microsoft-Defender-for-Identity-06.png]]

7. Enter **Access Key** and click on **Next**.
	![[0018 Onboarding-Microsoft-Defender-for-Identity-07.png]]

8. Once the installation is completed click on **Next**. 
	![[0018 Onboarding-Microsoft-Defender-for-Identity-08.png]]

## Setting Up Read-Only Account

1. Go to **Settings -> Directory Service Accounts**.
	![[0018 Onboarding-Microsoft-Defender-for-Identity-09.png]]

2. Make a read-only account in Windows Active Directory and enter these credentials inside Add credentials.
	![[0018 Onboarding-Microsoft-Defender-for-Identity-10.png]]

## Enabling Automatic Windows Auditing Configuration

1. Go to **Settings -> Identities**.
	![[0018 Onboarding-Microsoft-Defender-for-Identity-11.png]]

2. Go to **General -> Advanced Features** and enable **Automatic Windows Auditing Configuration**.
	![[0018 Onboarding-Microsoft-Defender-for-Identity-12.png]]

3. Click on **Save**.
	![[0018 Onboarding-Microsoft-Defender-for-Identity-13.png]]

## Conclusion

Once all these actions are completed the domain controllers are onboarded to Microsoft Defender for Identity. If your organization is using Microsoft Defender XDR and Microsoft highly recommends setting up Microsoft Defender for Identity as it will help with collecting signals to detect, investigate, and respond to complex cyberthreats.

## See also

- [[0052 Microsoft Defender for Identity Sensor v3.x RPC Audit Misconfigured.md|Microsoft Defender for Identity Sensor v3.x RPC Audit Misconfigured]]
- [[0055 Deep Dive Into Microsoft Defender for Identity Capabilities.md|Deep Dive Into Microsoft Defender for Identity Capabilities]]