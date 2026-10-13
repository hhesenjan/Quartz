---
title: "Using Device Control to Block USB Devices in MDE"
created: 2026-07-10
modified: 2026-07-10
tags: ["Windows", "MDE", "USB"]
draft: false
---

## Introduction

Many organizations nowadays are working on restricting the use of USB devices. In Microsoft Defender for Endpoint (MDE) its possible through Device Control as it comes with capabilities to block read, write, and execute permissions. In this article I'll go through implementing restriction for USB devices. 

## Purpose of Protecting USB Devices

**What is the purpose of protecting USB devices?** Many people uses USB devices to store images, videos, music, and in some situations cracked version of a paid application. An employee might also find a USB device at the doorstep of the office and decide to plug it into the company device which leads to threat actor gaining initial access to the network.

The whole purpose of protecting USB devices is to prevent scenarios such as these. I understand that many security professionals are in different situations so I would recommend them to analyze their own environment and find a middle ground which allows employees to be productive while being secure. In my situation it was to implement a restriction which allows users to only perform read and write to USB devices.

## Reusable Settings

Reusable Settings can be configured through Microsoft Intune. The purpose of it is to allow us to blacklist and whitelist USB devices, CD drives, SD cards, and etc... It can also be used for whitelisting only specific USB brands. In my situation I'll create a Reusable Settings which targets all USB devices using the RemovableMediaDevices attribute. 

1. Go to [Microsoft Intune](https://intune.microsoft.com/)

2. Click on **Endpoint Security -> Attack Surface Reduction Rules -> Reusable Settings**.
![[0049 Using Device Control to Block USB Devices in MDE 01.png]]

3. Click on **Add**.
![[0049 Using Device Control to Block USB Devices in MDE 02.png]]

4. Enter **Name and Description**.
![[0049 Using Device Control to Block USB Devices in MDE 03.png]]

5. Click on **Add -> Removable Storage**.
![[0049 Using Device Control to Block USB Devices in MDE 04.png]]

6. Click on **Configuration**.
![[0049 Using Device Control to Block USB Devices in MDE 05.png]]

7. Enter **RemovableMediaDevices** on both **Name and Primary ID**.
![[0049 Using Device Control to Block USB Devices in MDE 06.png]]

8. Review the configuration and create the reusable settings.
![[0049 Using Device Control to Block USB Devices in MDE 07.png]]


## Device Control

Device Control comes with many capabilities to manage bluetooth, usb devices, sd cards, and much more. In my situation I'll create a Device Control which specifically targets USB devices using Reusable Settings we made in the previous section.

1. Go to [Microsoft Intune](https://intune.microsoft.com/)

2. Click on **Endpoint Security -> Attack Surface Reduction Rules**.
![[0049 Using Device Control to Block USB Devices in MDE 08.png]]

3. Click on **Create -> Platform as Windows -> Profile as Device Control**.
![[0049 Using Device Control to Block USB Devices in MDE 09.png]]

4. Enter **Name and Description**.
![[0049 Using Device Control to Block USB Devices in MDE 10.png]]

5. In **Defender Section** use the following configurations.
![[0049 Using Device Control to Block USB Devices in MDE 11.png]]

6. In **Device Control Section** add the reusable settings by clicking on **Set Reusable Settings**.
![[0049 Using Device Control to Block USB Devices in MDE 12.png]]

7. Click on **Configure Access** and use the following configurations.<br>
    **Allow:** Read, Write, File Read, File Write
    
    **Deny:** Execute, File Execute, Print
![[0049 Using Device Control to Block USB Devices in MDE 13.png]]

8. In Microsoft Entra ID copy the **Object ID** of the group of users/computers to target.
![[0049 Using Device Control to Block USB Devices in MDE 14.png]]

9. Use the **Object ID** on the **SID and Computer SID**
![[0049 Using Device Control to Block USB Devices in MDE 15.png]]

10. Scroll down to **Connectivity** and set **Allow USB as Enabled**.
![[0049 Using Device Control to Block USB Devices in MDE 16.png]]

11. On **Assignment** select users that should receive the Device Control Policy.
![[0049 Using Device Control to Block USB Devices in MDE 17.png]]

12. Review the configuration and create Device Control policy.
![[0049 Using Device Control to Block USB Devices in MDE 18.png]]

I would recommend starting off the Device Control Policy with some test users instead of deploying it to the whole organization. It's also important to keep in mind that it might take anywhere from 15 minutes to 1 hour before the policy synchronzies with the device.

## Conclusion

Device Control Policy in Microsoft Defender for Ednpoint (MDE) allows us to manage USB devices, CD drives, and SD cards by ensuring that users can only perform specific actions. It allows us to build anywhere from simple configuration to block execute permission to a complex configuration which only allows specific USB brands. If you're using MDE, I would highly recommend spending some time setting up your Device Control Policy.

## See also

- [[0047 Automatically Update Apps through Intune (WinGet).md|Automatically Update APps through Intune (WinGet)]]
- [[0048 Disabling JavaScript and Flash Engine on Adobe Acrobat Reader.md| Disabling JavaScript and Flash Engine on Adobe Acrobat Reader]]
- [[0050 Disabling Autorun and Autoplay for Windows Devices.md|Disable Autorun and Autoplay for Windows Devices]]