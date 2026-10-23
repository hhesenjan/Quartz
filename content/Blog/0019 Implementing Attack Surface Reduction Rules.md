---
title: "Implementing Attack Surface Reduction Rules"
created: 2025-10-30
modified: 2025-10-30
tags: ["MDE", "ASR"]
draft: false
---

## Introduction

Attack Surface Reduction (ASR) is designed to limit the attack surface for Windows endpoints. While it's effective at blocking common attack techniques which are used by ransomwares and malwares, they may interfere with legitimate enterprise tools. In this article I'll go through the best practices to implement Attack Surface Reduction into your environment without creating any distruption to business.

## Implementation

Microsoft recommends implementing the Attack Surface Reduction (ASR) policy in auditing mode before enabling as they can create distruption throughout the business. Here's a overview of implementing Attack Surface Reduction in Auditing Mode:

1. Go to **Microsoft Intune -> Endpoint Security -> Attack Surface Reduction**.
    ![[0019 Implementing-Attack-Surface-Reduction-Rules-01.png]]

2. Click on **Create -> Select Windows -> Select Attack Surface Reduction**
    ![[0019 Implementing-Attack-Surface-Reduction-Rules-02.png]]

3. Enter **Name and Description** for Attack Surface Reduction policy.
    ![[0019 Implementing-Attack-Surface-Reduction-Rules-03.png]]

4. Select all features to enable in audit mode.
    ![[0019 Implementing-Attack-Surface-Reduction-Rules-04.png]]

5. Select **Scope Tag**.
    ![[0019 Implementing-Attack-Surface-Reduction-Rules-05.png]]

6. Select **All Devices/Group** to onboard to ASR policy.
    ![[0019 Implementing-Attack-Surface-Reduction-Rules-06.png]]

7. **Review Configuration** and click on **Create**. 
    ![[0019 Implementing-Attack-Surface-Reduction-Rules-07.png]]

## Auditing

When the Attack Surface Reduction policy is created, it's recommended to start auditing the programs that could be affected by Attack Surface Reduction rules.

1. Go to **Microsoft Defender 365 -> Reports -> Attack Surface Reduction**.
    ![[0019 Implementing-Attack-Surface-Reduction-Rules-08.png]]

2. Monitor `Blocked/Adited` field to monitor actions ASR would perform. 
    ![[0019 Implementing-Attack-Surface-Reduction-Rules-09.png]]

## Conclusion

Attack Surface Reduction is great for protecting the organization from common attack techniques. However, it could end up blocking legitimate enterprise applications therefore it's recommended to enable it in auditing mode and slowly enable the different features.