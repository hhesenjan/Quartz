---
title: "Adding IoCs to Microsoft Defender XDR"
created: 2026-02-23
modified: 2026-02-23
tags: ["MDE", "SIEM"]
draft: false
---

## Introduction

Microsoft Defender XDR has a Indication of Compromise (IoC) list which we can use to allow, warn, and block depending on the File Hash, IP-Address, URLs, and Certificates. In this article I'll go through finding IoC page and whitelist a file hash.

## IoCs Page

This section goes through finding the Microosft Defender XDR IoCs.

1. Go to [Microsoft Defender Portal](https://security.microsoft.com/)

    ![[0035 Adding-IoCs-to-Microsoft-Defender-XDR-01.png]]

2. Click on **Endpoints**

    ![[0035 Adding-IoCs-to-Microsoft-Defender-XDR-02.png]]

3. Go to **Indicators**

    ![[0035 Adding-IoCs-to-Microsoft-Defender-XDR-03.png]]

## Adding File Hash Into IoCs

So basically there is a issue with TeamViewer where it fails to update itself and triggers false positive alerts. Here is overview of the TeamViewer Hash.e
* File Hash : `48599dcb6e68b95eabc3f1fae33ad44c39a1ea189ae14350fa6e294ebea4d5ae`

In this section I'll go through whitelisting that SHA-256 in Microsoft Defender XDR to stop the false positive security alerts that we have been receiving recently.

1. Click on **Add item**

    ![[0035 Adding-IoCs-to-Microsoft-Defender-XDR-04.png]]

2. Enter the **File Hash, Title, and Description** then click **Next** 

    ![[0035 Adding-IoCs-to-Microsoft-Defender-XDR-05.png]]

3. Click on **Allow** and **Next**

    ![[0035 Adding-IoCs-to-Microsoft-Defender-XDR-06.png]]

4. On **Organizational Scope** click on **Next**

    ![[0035 Adding-IoCs-to-Microsoft-Defender-XDR-07.png]]

5. Review the IoC and click on **Submit**

    ![[0035 Adding-IoCs-to-Microsoft-Defender-XDR-08.png]]

We should no longer receive false positive security alerts related to that file. However, if you do still receive false positive security alerts I recommend closing down these security alerts as false positive as these metrics are used to train up Microsoft Defender. 

## Conclusion

Microsoft Defender XDR is no doubt one of the most powerfull XDR systems out there but sometimes finding features can be difficult because it comes with so many features. Hopefully, this post has helped you with finding the correct place and reduce the false positive alerts that you have been receiving from Microsoft Defender XDR.