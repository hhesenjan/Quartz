---
title: "Troubleshoot MDE Onboarding Issues"
created: 2026-06-19
modified: 2026-06-19
tags: ["Windows", "MDE"]
draft: false
---

## Introduction

So basically while onboarding Microsoft Defender for Endpoint (MDE) there can be some issues with some endpoints failing to be onboarded because of missing components or broken components. In this article I'll go through fixing broken/missing components.

## Solution

All these commands should fix the missing and broken components related to Windows which then should allow Microsoft Defender for Endpoint (MDE) to run without any issues. 

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

What all these commands does is updating the operating system and installing/fixing the different components which Microsoft Defender for Endpoint uses. 

## Conclusion

Microsoft Defender for Endpoint (MDE) components are a-part of Windows operating system and sometimes these components can be missing or broken because of Windows Updates. However, after using the commands mentioned in the solution it should solve these issues. 
