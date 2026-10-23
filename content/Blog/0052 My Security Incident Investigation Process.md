---
title: "My Security Incident Investigation Process"
created: 2026-08-07
modified: 2026-08-07
tags: ["Personal"]
draft: true
---

## Introduction

So basically, I thought I would write about the way I investigate security incidents so I can look back in the future and see the improvements I made throughout the years. Overall the point of the blog is too see the way I have developed throughout the years. Anyway lets dive into the way I investigate security incidents.

## Telemetry Data

**You cannot protect something you don't know anything about.** I understand that many organizations doesn't want to spend money on collecting telemetry data from their endpoints, servers, and cloud resources. It's crucial to collect telemetry data since when a security incidnet occurs where company data is compromised, the source of truth will be the telemetry data. **It might sound expensive to spend \$2000/m on storing telemetry data but the real expense comes when the company is compromsied and there is no telemetry data to follow.**

**Learn the tools in your toolbox.** As a cybersecurity professional it's crucial to understand the tools in your toolbox. As an example if your organization uses CrowdStrike learning CQL can help you significantly during a security incidents and potentially save you hours.

## Filtering Telemetry Data

**We humans are not meant to look through telemetry data.** I understand that there are some people who does love to look through telemetry data but the point of telemetry data is to retrieve the data that is important for us. And using that telemetry data to visualize it in a pattern to understand the security incident further. As an example a employee usually logins from `161.110.112.10` but one day logins from `161.33.110.15` and then starts performing a port scan.

**Focus on something specific.** Instead of focusing on multiple of things such as user identity, computer identity, and IPv4 address instead focus on a single target and try to collect as much information about it. Once that is completed move onto the next thing to focus on that you believe is important. 

## Offensive Security

**Learn offensive security to understand threat actors.** I believe its important for us as cybersecurity professionals to learn the tools and techniques the threat actors uses to compromise organizations. As an example when a threat actor obtains access to the internal environment attacks such as password spray, kerberoasting, dumping ldap, and accessing SMB services is quite common.

## Affected Resources

**Communication is extremely important.** Once all the information is collected we should commnunicate with the owners of the assets. This is probbaly where my weakness occurs since I tend to be too focused on the data and might forget to inform the asset owners since I became too focus on obtaining all the informations.
## Canvas

**Make the telemetry data and other information readable.** I would recommend creating a canvas using obsidian and including relevant KQLs, usernames, computer names, IPv4 addresses, and so forth. As this can help with explaining the situation and showing what really happened when the asset owners wants to dive more into it.

## Conclusion

I mainly focus on Offensive Security at my free time since I find it interesting to find a way to break something that works fine. I think these skills has come in good use for me since it allows me to map the data the threat actor might have managed to obtain and what asset could potentially be infected by them. And note taking apps like obsidian has allowed me to map it in a way others can understand what happened. 