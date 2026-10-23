---
title: "Implementing Microsoft Defender SmartScreen"
created: 2025-11-05
modified: 2025-11-05
tags: ["MDE", "SMARTSCREEN"]
draft: false
---

## Introduction

Microsoft Defender SmartScreen is desinged to protect endpoints from malicious websites, applications, and files by displaying an early warning before execution. It comes with anti-phishing, anti-malware, reputation-based url, and app protection to detect malicious applications.

## Implementation

1. Go to **Microsoft Intune -> Endpoint Protection -> Attack Surface Reduction**.
    ![[0021 Implementing-Microsoft-Defender-SmartScreen-01.png]]

2. Enter **Name and Description**.
    ![[0021 Implementing-Microsoft-Defender-SmartScreen-02.png]]

3. Select **Scope Tags**.
    ![[0021 Implementing-Microsoft-Defender-SmartScreen-03.png]]

4. Select **All Devices**.
    ![[0021 Implementing-Microsoft-Defender-SmartScreen-04.png]]

5. Review **Configuration** and click on **Create**.
    ![[0021 Implementing-Microsoft-Defender-SmartScreen-05.png]]

> [!IMPORTANT]+ IMPORTANT
> Once the Application Control Policy is applied on the endpoints it will force the endpoints to reboot within 10 minutes otherwise it will automatically reboot the system.

## Conclusion

Microsoft Defender SmartScreen is exceptional for warning about malicious websites, applications, and files. All organizations should enable Microsoft Defender SmartScreen to warn their users and protect organizational assets from threat actors.