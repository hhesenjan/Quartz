---
title: "Implementing Microsoft Defender Antivirus Policy for Linux Systems"
created: 2026-04-06
modified: 2026-04-06
tags: ["MDE", "LINUX"]
draft: false
---

## Introduction

While Microsoft Defender for Endpoint comes with default configurations which can be assigned to Windows Machines. There aren't many writeups on setting up Antivirus Policy for Linux servers. In this article I'll go through implementing Microsoft Antivirus Policy for Linux systems.

## Enforcement Scope

1. Go to **Microsoft Defender -> Settings -> Endpoints**

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-01.png]]

2. Enable **Use MDE to enforce security configuration settings from Intune** 

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-02.png]]

3. Select **Linux Devices** and **On tagged devies**

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-03.png]]

4. Click on **Save** after enabling all these settings.

## Adding Device Tag

1. Find the Linux system on MDE.

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-04.png]]

2. Click on **...** and **Manage Tags**

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-05.png]]

3. Add the tag `MDE-Management`

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-06.png]]

## Antivirus Policy

1. Go to **Endpoint Security -> Antivirus**.

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-07.png]]

2. Click on **Create** and select **Linux** as **Platform** and **Profile** as **Microsoft Defender Antivirus**.

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-08.png]]

3. Enter the **Name** and **Description**.

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-09.png]]

4. In **Antivirus Engine** use the following baseline configuration.

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-10.png]]

5. In **Antivirus Engine** use the following baseline configuration.

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-11.png]]

6. When it comes to **Threat Type Settings** I would recommend using **Audit Mode** to see if any false positives occurs within your environment.
    
    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-12.png]]

7. In **Scanning Options** I decided to enable more strict features since the system administrator wanted to see how much it would affect the performance.

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-13.png]]

8. In **Network Protection** I recommend enabling **Auditing Mode** so we can monitor what operations would be blocked and allowed.

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-14.png]]

9. Select your scope tag.

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-15.png]]

10. Select all devices.

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-16.png]]

11. Review the configurations and click **Save**.

    ![[0039 Implementing-Microsoft-Defender-Antivirus-Policy-for-Linux-17.png]]

All devices with the `MDE-Management` tag will automatically be assigned the Antivirus Policy without them being onboarded to Microsoft Defender for Cloud.

## Conclusion

Microsoft Defender XDR comes with a-lot of features. I originally thought it wouldn't be possible to onboard Antivirus Policy on Linux systems since these aren't onboarded through Microsoft Intune. However, using **Enforcement Scope** and **Tagging** it's possible to enable Antivirus Policy on Linux systems.
