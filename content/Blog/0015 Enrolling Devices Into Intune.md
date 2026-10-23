---
title: "Enrolling Windows Device Into Intune"
created: 2025-10-01
modified: 2025-10-01
tags: ["INTUNE", "M365"]
draft: false
---

## Introduction

Microsoft Intune is the modern way of managing devices. It also comes with capabilities for us to automate tasks which the IT engineers are doing manually. In this article I'll go through multiple of ways to enroll a Windows device into Microsoft Intune.

## OOBE Enrollment

We can enroll a device into Microsoft Intune in the OOBE stage with the following steps.

1. Open `cmd.exe` with `SHIFT + F10`.
2. Use `powershell.exe -ep bypass` inside the terminal.
3. Use `Install-Script Get-WindowsAutopilotInfo -Force` to download the Windows Autopilot script.
4. Use `Get-WindowsAutopilotInfo -Online -GroupTag NO1` to enroll the device into Microsoft Intune.
5. Use your company credentials at login prompt.
6. Use `shutdown -r -t 0` to reboot the system.

Once these steps are completed a Microsoft login page with the company logo should come up. Where the user can login to enroll the laptop into Microsoft Intune.

## Hash Extraction Enrollment

We can also enroll a Windows device into Microsoft Intune by extracting the laptop hash.

1. Attach a USB into the Windows device.
2. Open `cmd.exe` with `SHIFT + F10`.
3. Use `Install-Script Get-WindowsAutopilotInfo -Force` to download the Windows Autopilot script.
5. Use `Get-WindowsAutopilotInfo -GroupTag <TAG> -OutputFile D:\SERIALNUMBER.csv` to extract hash.
6. Upload the `SERIALNUMBER.csv` through [Windows Autopilot Devices](https://intune.microsoft.com/#view/Microsoft_Intune_Enrollment/AutopilotDevices.ReactView/filterOnManualRemediationRequired~/false).
	![[0015 Enrolling-Devices-Into-Intune-01.png]]
7. Wait 15 minutes and reboot the system.

After rebooting a Microsoft login page with the company logo should come up and from there the user can start logging in and start enrolling the Windows device into our Microsoft Intune environment. 

## Troubleshooting Issues

If you experience any issues during the enrollment of these PCs, I recommend checking out the following file:

* `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\AppWorkload.log`

## Conclusion

The [[0015 Enrolling Devices Into Intune.md#OOBE Enrollment]] is the easiest and most efficient way of uploading a laptop hash into Microsoft Intune. However, the [[0015 Enrolling Devices Into Intune.md#Hash Extraction Enrollment]] is also great for uploading laptop hashes into Microsoft Intune especially if you don't have physical access to the system as you can request the user to execute these commands and send the hash file to us.