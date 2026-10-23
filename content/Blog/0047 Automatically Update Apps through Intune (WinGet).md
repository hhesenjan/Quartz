---
title: "Automatically Update Apps through Intune (WinGet)"
created: 2026-06-26
modified: 2026-06-26
tags: ["Intune", "Windows", "WinGet"]
draft: false
---

## Introduction

Microsoft Intune is a greay way of managing devices to ensure employees receives the necessary applications and policies to be productive and secure. It also helps with reducing the time IT-Support would spend on setting up the device. However, one of the biggest issues with Microsoft Intune is updating the application already installed on the system since a new application profile has to be created and tested which takes a-lot of time.

Instead of creating a application profiles every time an application needs to be updated we can instead perform automatic updates using WinGet. In this article I'll be digging into updating applications on your devices without purchasing an expensive product.

## Understanding WinGet

WinGet is a package manager that comes with Windows and it allows us to install and update applications through the command line. This saves us a-lot of time since if we were going to install multiple of applications we would need to go to multiple of website and then click on multiple of buttons. Instead of doing all these things we can instead use WinGet to install the application. 

```powershell title="WinGet: Visual Studio"
# Install Visual Studio using WinGet
winget install -e --id Microsoft.VisualStudioCode

# Update Visual Studio using Winget
winget update -e --id Microsoft.VisualStudioCode
```

You can view all the application that can be installed with WinGet through the following site [WInstall](https://winstall.app/).  I usually perform a factory reset once a year and WinGet has been a life saver since it allows me to install all the applications I use day-to-day in seconds.

## Updating Application

WinGet also comes with the capability to update applications one-by-one or updating all of them. This allows us to update applications on the devices without spending thousands of dollars on patch management solution which synchronizes with Microsoft Intune.

```powershell title="WinGet: One-by-One or All"
# Get list of applications.
winget list

# Update specific application on device.
winget upgrade google.chrome --silent --accept-source-agreements --accept-package-agreements

# Update all applications on device.
winget upgrade --all --silent --accept-source-agreements --accept-package-agreements
```

We can execute the update command on our users devices using Remediation Scripts in Microsoft Intune. I suggest running the command once a week in the early morning while employees are socializing with their colleagues. 


> [!IMPORTANT]- Legacy Applications
> WinGet with `--all` parameter will update all the application installed on the device. If an employee requires older version of a appplication to perfomr their work it can ruin their productivity therefore it's crucial to understand your environment before updating everything on your users system.

## Microsoft Intune Rollout

This section of the article goes through updating all the application using WinGet using Remediation Scripts.

1. Go to [Microsoft Intune](https://intune.microsoft.com/).
2. Click on **Devices -> Remediation Script**.
    ![[0047 Automatically Update Apps through Intune (WinGet) 01.png]]

3. Click on **"Create"**.
    ![[0047 Automatically Update Apps through Intune (WinGet) 02.png]]

4. Enter **Name and Description**.
    ![[0047 Automatically Update Apps through Intune (WinGet) 03.png]]
    
5. Here is the **Detection and Script** to use.
    ```powershell title="Detection Script"
        $Output = winget.exe upgrade

        if ($Output -like "*upgrades available*") {
            Exit 1
        } else {
            Exit 0
        }
    ```
    ```powershell title="Automatic Update"
        $arguments = @("upgrade", "--all", "--silent", "--accept-source-agreements", "--accept-package-agreements")
        Start-Process -FilePath "winget.exe" -ArgumentList $arguments -Wait -NoNewWindow
    ```
    ![[0047 Automatically Update Apps through Intune (WinGet) 04.png]]

5. Choose **All Users** or specific users to test the remediation script on. 
    ![[0047 Automatically Update Apps through Intune (WinGet) 05.png]]

6. Create the Remediation Script after reviewing it.
    ![[0047 Automatically Update Apps through Intune (WinGet) 06.png]]

## Conclusion

I understand that many companies are using third party tools to perform automatic updates on their users devices which costs hundreds to thousands of dollars. Instead of spending that much money on updating application on our users devices we can instead use WinGet and automatically update our users devices every day or every week.

I strongly believe it's crucial for us to use the tools that are already available for us instead of purchasing a new product that will cost the company money especially as there is uncertaincy happening around the world.

## See also

- [[0048 Disabling JavaScript and Flash Engine on Adobe Acrobat Reader.md| Disabling JavaScript and Flash Engine on Adobe Acrobat Reader]]
- [[0049 Using Device Control to Block USB Devices in MDE.md|Using Device Control to Block USB Devices in MDE]]
- [[0050 Disabling Autorun and Autoplay for Windows Devices.md|Disable Autorun and Autoplay for Windows Devices]]