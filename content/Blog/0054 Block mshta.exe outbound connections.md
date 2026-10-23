---
title: "Block MSHTA.EXE Outbound Connections"
created: 2026-08-28
modified: 2026-08-28
tags: ["MDE", "HARDENING"]
draft: false
---

## Information

Microsoft HTML Host Application (mshta.exe) is a legacy feature in Windows that runs HTML Application (HTA) files. These HTA files can consists of malicious VBScript that steals data or returns a reverse shell to the threat actor. This is why Microsoft recommends us to disable outbound connections for mshta.exe process. In this article I'll go through disabling outbound connections for mshta.exe process.

## Legacy Applications

I work in the offshore industry where small changes can create massive distruptions throughout the busienss. This is why I would highly recommend you to research your own environment before disabling outbound connection for the mshta.exe process. Here are some KQLS that could be useful to see whether mstha.exe is widely used in your organization:

```sql
DeviceNetworkEvents
| where Timestamp > ago(30d)
| where InitiatingProcessFileName =~ "mshta.exe"
| summarize Connections=count(), Devices=dcount(DeviceName),
            FirstSeen=min(Timestamp), LastSeen=max(Timestamp)
          by RemoteUrl, RemoteIP, RemotePort, RemoteIPType
| order by Connections desc
```

```sql
DeviceProcessEvents
| where Timestamp > ago(90d)
| where FileName =~ "mshta.exe"
| extend HtaPath = extract(@'([a-zA-Z]:\\[^"]+\.hta|\\\\[^"]+\.hta)', 1, tolower(ProcessCommandLine))
| where isnotempty(HtaPath)
| summarize Executions=count(), Devices=dcount(DeviceName) by HtaPath
| order by Devices desc
```

These two queries helped me with understanding my own environment sicne I found out that a user is using a legacy application which requires the HTA files to be ran with outbound connections.

## Implementation

1. Go to [**Microsoft Intune**](https://intune.microsoft.com/).
2. Click on **Endpoint Security -> Firewalls**.
    ![[0054 Block MSHTA.EXE Outbound Connections 01.png]]
3. Click on **Create Policy**.
    ![[0054 Block MSHTA.EXE Outbound Connections 02.png]]
4. Select Platform as **Windows** and Profile as **Windows Firewall Rules**.
    ![[0054 Block MSHTA.EXE Outbound Connections 03.png]]
5. Enter **Name** and **Description**.
    ![[0054 Block MSHTA.EXE Outbound Connections 04.png]]
6. Create two firewall rules with the following naming conversions.
    ![[0054 Block MSHTA.EXE Outbound Connections 05.png]]
7. Use the following configurations on the different settings.
    
    **Block mshta.exe on 32-bit**
    - Enabled: Enabled
    - Interface Types: All
    - File Path: `%SystemRoot%\System32\mshta.exe`
    ![[0054 Block MSHTA.EXE Outbound Connections 06.png]]

    **Block mshta.exe on 64-bit**
    - Enabled: Enabled
    - Interface Types: All
    - File Path: `%systemroot%\syswow64\mshta.exe`
    ![[0054 Block MSHTA.EXE Outbound Connections 07.png]]
8. On the Assignment section use **All Users**.
    ![[0054 Block MSHTA.EXE Outbound Connections 08.png]]

9. Review the **Windows Firewall Configuration** and **Create It**.
    ![[0054 Block MSHTA.EXE Outbound Connections 09.png]]

## Conclusion

Microsoft HTML Application Host (mshta.exe) is a legacy feature that comes with latest Windows operating systems since many applications still uses it for some background tasks. If your organization is not using HTML Application File in any form, I would highly recommend blocking outbound connections since it's commonly used by threat actors to execute malicious code which returns reverse shells to the threat actor command and control center. 

Additionally, it's a best practise to block features and things your organization isn't using because anything that is left open can be abused by threat actors.