---
title: "Enable Auto-Expanding Archiving for Exchange Online Mailbox"
created: 2025-12-15
modified: 2025-12-15
tags: ["M365", "EXCHANGE"]
draft: false
---

## Introduction

The Chief Commerical Officer (CCO) recently reported to me that their Exchange Online archive storage was approaching 100GB and requested an increase. The issue was resolved by enabling Auto-Expanding Archiving which is included in Microsoft E5, Microsoft A3, and Microsoft E3 license. 

This article walks through enabling Auto-Expanding Archiving feature for a single user to remove Exchange Online archiving limits.

## Enabling Auto-Expanding Archive

You will need the Exchange Online Administrator role in Microsoft 365 environment to enable Auto-Expanding Archiving feature.

1. Open PowerShell and Install [Microsoft Exchange Online Module](https://learn.microsoft.com/en-us/powershell/exchange/connect-to-exchange-online-powershell?view=exchange-ps).
    ```powershell
    Install-Module ExchangeOnlineManagement
    ```
2. Reopen PowerShell and connect to Microsoft Exchange Online.
    ```powershell
    Connect-ExchangeOnline
    ```
3. Login with the account that has **Exchange Administrator** role.
    ![[0028 Enabling Auto-Scaling-for-Exchange-Online-Archive-01.png]]

4. Use this command to see if Auto-Expanding Archiving is enabled on their account.
    ```powershell {"<EMAIL>"}
    Get-Mailbox <EMAIL> | FL AutoExpandingArchiveEnabled
    ```

5. Use this command to enable Auto-Expanding Archiving for the user.
    ```powershell {"<EMAIL>"}
    Enable-Mailbox <EMAIL> -AutoExpandingArchive
    ```

After enabling the Auto-Expanding Archiving feature it can take 1 hour for the change to take effect. However, once the change is applied the user should be able to store 1.5 TB of archive emails.

## Conclusion

The Chief Executive Officer, Chief Financial Officer, and Chief Commercial Officer typically recieves a-lot of emails and the archive storage allows them to store old emails without deleting them. When the emails are stored in the archive folder they are only accessible when they are online and this helps with combating caching issues that commonly occurs with the Outlook client.

The executives can quickly fill up the archive storage therefore eabling Auto-Expanding Archiving on their account is a great move as that will allow them to store emails up to 1.5 TB.