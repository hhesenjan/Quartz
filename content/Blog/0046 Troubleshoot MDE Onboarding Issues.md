---
title: "Troubleshoot MDE Onboarding Issues"
created: 2026-06-19
modified: 2026-06-19
tags: ["Windows", "MDE"]
draft: false
---

## Introduction

So basically some devices that where offshore were failing to be onboarded to Microsoft Defender for Endpoint (MDE) after running the MDE onboarding script. I originally taught it was because of brandwidth limitations but after a week these endpoints where still not being onboarded therefore I decided to troubleshoot the issue and after spending hours troubleshooting I found the solution.

In this article I'll go through the way I solved the issue to hopefully help others who are in similar position.

## Solution

In some cases a device that is offshore might have failed to update a file or component since it lost internet connectivity. And these files and components might be used by MDE to onboard the device to MDE.

```powershell title="Solution"
# 1. Perform Windows Update (Wait Around 3 minutes)
wuauclt /detectnow /updatenow

# 2. Reboot the endpoint
shutdown -r -t 0

# 3. Installing Microsoft Defender for Endpoint Sense sensor
Add-WindowsCapability -Online -Name "Microsoft.Windows.Sense.Client~~~~"

# 4. Repairing any issues with Windows
DISM /Online /Cleanup-Image /RestoreHealth

# 5. Repairing any missing files in WIndows
sfc /scannow
```

What all these commands does is update the operating system, installing the sense censor, and fix the corrupted component and files. Once all these actions are performed I would highly recommend running the MDE onboarding script again.

## Conclusion

So basically some devices that where offshore were failing to be onboarded to Microsoft Defender for Endpoint (MDE) because some component and files where corrupted. And using the commmands that where mentioned in the [[#Solution|Solution]] should fix issue with the operating system.
