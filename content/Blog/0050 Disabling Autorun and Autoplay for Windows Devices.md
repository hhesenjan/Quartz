---
title: "Disable Autorun and Autoplay for Windows Devices"
created: 2026-07-18
modified: 2026-07-18
tags: ["Windows", "MDE", "PowerShell"]
draft: false
---

## Introduction

One of Microsoft Security recommendations is to disable Autorun and Autoplay on USB and MTP devices. The reason Microsoft recommends it is because threat actors could abuse these capabilities to autorun a malware or ransomware. Here is a quick overview of these security recommendations:

- Set default behavior for 'AutoRun' to 'Enabled: Do not execute any autorun commands'
- Disable 'Autoplay for non-volume devices'
- Disable 'Autoplay' for all drives

In this article I'll be going through implementing these security recommendations from Microsoft. Also implementing these security recommendations can help with increasing your secure score by 0.5% to 1% and who doesn't want that...

## PowerShell

I understand that in my recent articles I have been building PowerShell scripts to harden the different systems instead of configuring it through Microsoft Intune. The reason for that is because I work in the offshore industry sector where the endpoints are managed through SCCM and NinjaOne. Here is the PowerShell script I built which will implement these security recommendations:

```powershell title="DisableAutorunAndAutoplay.ps1"
#
# Configuration Name: Set default behavior for 'AutoRun' to 'Enabled: Do not execute any autorun commands'
# Configuration ID: SCID-70
#
function DisableUSBAutorun() {
    $explorer_reg_path = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer"

    Write-Host "[#] Running DisableUSBAutorun function."
    if (-not (Get-ItemProperty -Path $explorer_reg_path -Name "NoAutorun" -ErrorAction SilentlyContinue)) {
        Write-Host "`t[#] Creating `NoAutorun` registry with value 1."
        New-ItemProperty -Path $explorer_reg_path -Name "NoAutorun" -Value 1 | Out-Null
    }
    elseif ((Get-ItemProperty -Path $explorer_reg_path -Name "NoAutorun" -ErrorAction SilentlyContinue)){
        Write-Host "`t[#] Modifying NoAutorun to value 1."
        Set-ItemProperty -Path $explorer_reg_path -Name "NoAutorun" -Value 1 | Out-Null
    }
    else {
        Write-Host "[!] An unknown issue occurred while trying to create or modify the value."
    }

    Write-Host "[#] Exiting DisaleUSBAutorun function."
}

#
# Configuration Name: Disable 'Autoplay' for all drives
# Configuration ID: SCID-69
#
function DisableAutoplayMTP() {
    $explorer_reg_path = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer"

    Write-Host "[#] Running DisableAutoplayMTP() function."
    if (-not (Get-ItemProperty -Path $explorer_reg_path -Name "NoDriveTypeAutoRun" -ErrorAction SilentlyContinue)) {
        Write-Host "`t[#] Creating `NoDriveTypeAutoRun` registry with value 1."
        New-ItemProperty -Path $explorer_reg_path -Name "NoDriveTypeAutoRun" -Value 255 | Out-Null
    }
    elseif ((Get-ItemProperty -Path $explorer_reg_path -Name "NoDriveTypeAutoRun" -ErrorAction SilentlyContinue)){
        Write-Host "`t[#] Modifying `NoDriveTypeAutoRun` to value 1."
        Set-ItemProperty -Path $explorer_reg_path -Name "NoDriveTypeAutoRun" -Value 255 | Out-Null
    }
    else {
        Write-Host "[!] An unknown issue occurred while trying to create or modify the value."
    }

    Write-Host "[#] Exiting DisableAutoplayMTP() function."
}

#
# Configuration Name: Disable 'Autoplay for non-volume devices'
# Configuration ID: SCID-67
#
function DisableAutoplayNonVolumeDevices() {
    $explorer_reg_path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\Explorer"

    Write-Host "[#] Running DisableAutoplayNonVolumeDevices() function."
    if (-not (Get-ItemProperty -Path $explorer_reg_path -Name "NoAutoplayfornonVolume" -ErrorAction SilentlyContinue)) {
        Write-Host "`t[#] Creating `NoAutoplayfornonVolume` registry with value 1."
        New-ItemProperty -Path $explorer_reg_path -Name "NoAutoplayfornonVolume" -Value 1 | Out-Null
    }
    elseif ((Get-ItemProperty -Path $explorer_reg_path -Name "NoAutoplayfornonVolume" -ErrorAction SilentlyContinue)){
        Write-Host "`t[#] Modifying `NoAutoplayfornonVolume` to value 1."
        Set-ItemProperty -Path $explorer_reg_path -Name "NoAutoplayfornonVolume" -Value 1 | Out-Null
    }
    else {
        Write-Host "[!] An unknown issue occurred while trying to create or modify the value."
    }
    
    Write-Host "[#] Exiting DisableAutoplayNonVolumeDevices() function."
}

#
# Purpose -
# Reverts the changes applied to the different registry values.
function RevertChanges() {
    $explorer_reg_path_01 = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer"
    $explorer_reg_path_02 = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\Explorer"

    Write-Host "[#] Running RevertChanges() function."
    if ((Get-ItemProperty -Path $explorer_reg_path_01 -Name "NoAutorun" -ErrorAction SilentlyContinue).NoAutorun -eq 1) {
        Write-Host "`t[#] Changing `NoAutorun` registry to value 0."
        Set-ItemProperty -Path $explorer_reg_path -Name "NoAutorun" -Value 0
    } 

    if ((Get-ItemProperty -Path $explorer_reg_path_01 -Name "NoDriveTypeAutoRun" -ErrorAction SilentlyContinue).NoDriveTypeAutoRun -eq 255) {
        Write-Host "`t[#] Changing `NoDriveTypeAutorun` registry to value 145"
        Set-ItemProperty -Path $explorer_reg_path -Name "NoDriveTypeAutoRun" -Value 145
    }

    if ((Get-ItemProperty -Path $explorer_reg_path_02 -Name "NoAutoplayfornonVolume" -ErrorAction SilentlyContinue).NoAutoplayfornonVolume -eq 1) {
        Write-Host "`t[#] Changing `NoAutoplayfornonVolume` registry to value 145"
        Set-ItemProperty -Path $explorer_reg_path -Name "NoDriveTypeAutoRun" -Value 0
    }

    Write-Host "[#] Exiting RevertChanges() function."
}

#
# Purpose -
# Runs DisableUSBAutorun and DisableMTPAutorun
#
function main() {
    $explorer_reg_path_01 = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer"
    $explorer_reg_path_02 = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\Explorer"

    if ((Test-Path -Path $explorer_reg_path_01) -eq $true) {
        DisableUSBAutorun
        DisableAutoplayMTP
    }
    
    if ((Test-Path -Path $explorer_reg_path_02) -eq $true)
    {
        DisableAutoplayNonVolumeDevices
    }
}

main
```

The `DisableUSBAutorun` function will disable autorun capabilities for USB devices. The `DisableAutoplayMTP` and `DisableAutoplayNonVolumeDevices`  will disable autoplay capabilities in Windows. While the `RevertChanges` function is responsible for reverting the changes applied incase of a disaster recovery situation.

```powershell title="DetectionScript.ps1"
#
# Purpose -
# Microsoft Intune script to detect whether the implementations where successfully applied.
# 
function Detection() {
    $explorer_reg_path_01 = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer"
    $explorer_reg_path_02 = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\Explorer"
    $success = $false

    if (((Get-ItemProperty -Path $explorer_reg_path_01 -Name "NoAutorun" -ErrorAction SilentlyContinue).NoAutorun -eq 1) -and ((Get-ItemProperty -Path $explorer_reg_path_01 -Name "NoDriveTypeAutoRun" -ErrorAction SilentlyContinue).NoDriveTypeAutoRun -eq 255) -and ((Get-ItemProperty -Path $explorer_reg_path_02 -Name "NoAutoplayfornonVolume" -ErrorAction SilentlyContinue).NoAutoplayfornonVolume -eq 1)) 
    {
        Write-Host "Settings successfully applied" 
        Exit 0
    }
    Exit 1
}

Detection
```

The `Detection()` function is responsible for detecting whether the configuration applied successfully to the Windows device which is useful in scenarios where you will be using remediation scripts to run the script which disables autorun and autoplay. 

## Remediation Scripts

You should now be able to harden the Windows devices by running the PowerShell scripts through Remediation Script option in Microsoft Intune. The `DisableAutorunAndAutoplay.ps1` can be uploaded as **Remediation Script File** and `DetectionScript.ps1` can be uploaded as **Detection Script File**. If you need step-by-step tutorial I would recommend reading through previous articles [[0048 Disabling JavaScript and Flash Engine on Adobe Acrobat Reader.md#Remediation-Script|Remediation Script]].

## Conclusion

I would highly recommend implementing these changes since it prevents an employee and user from accidentally running a malicious file which is ran through autorun and autoplay features. Disabling Autorun and Autoplay features for Windows devices should also gain your organization 0.5 to 1% higher secure score which are good for compliance reasons. Hopefully, this article has helped you with increasing your organization's security posture.

## See also

- [[0047 Automatically Update Apps through Intune (WinGet).md|Automatically Update APps through Intune (WinGet)]]
- [[0048 Disabling JavaScript and Flash Engine on Adobe Acrobat Reader.md| Disabling JavaScript and Flash Engine on Adobe Acrobat Reader]]
- [[0049 Using Device Control to Block USB Devices in MDE.md|Using Device Control to Block USB Devices in MDE]]
