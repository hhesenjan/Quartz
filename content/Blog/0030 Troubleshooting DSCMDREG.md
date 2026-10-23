---
title: "Troubleshooting Microsoft Entra ID Devices"
created: 2026-01-07
modified: 2026-01-07
tags: ["INTUNE"]
draft: false
---

## Information

When a device is onboarded to Microsoft Intune an object is created in Microosft Entra ID. The object contains a-lot of information related to the device such as Device State, Device Details, Tenant Details, User State, SSO State, and Diagnostics Data.

The `dscmdreg` enables us to view the configurations that is applied to the device to troubleshoot the issues that are ocurring with SSO and so forth...

## Device State

In Microsoft Entra ID an device can be assigned different device states depending on the way Microsoft Intune environment is configured.

| AzureAdJoined	| EnterpriseJoined | DomainJoined | Device state                     | 
| ------------- | ---------------- | ------------ | -------------------------------- |
| YES	          | NO               | NO           |	Microsoft Entra joined           |
| NO	          | NO               | YES          |	Domain Joined                    |
| YES           | NO               | YES          |	Microsoft Entra Hybrid Joined    |
| NO            | YES	             | YES	        | On-premises DRS Joined           |

An example if a user is failing to login to cloud enterprise applications it oculd be related to the device being only domain joined instead of being Microsoft Entra Joined. 

## Device Info

The `dsregcmd /status` command will show all the information about the device that is in the Microsoft Entra ID environment. Additionally, it will provide us with information if there are any issues between device and Microsoft Entra ID.

```powershell {6-8}
C:\Users\husenjan> dsregcmd /status
+----------------------------------------------------------------------+
| Device State                                                         |
+----------------------------------------------------------------------+

             AzureAdJoined : YES
          EnterpriseJoined : NO
              DomainJoined : YES
                DomainName : HUSENJAN
           Virtual Desktop : NOT SET
               Device Name : NO-PC001.int.husenjan.com
```

With all these informations we can view misconfigurations and potentially find the issue that is ocurring between the device and Microsoft Entra ID environment.

## Leaving & Joining

In some circumstances the device might need to be unjoined and rejoined from Microsoft Entra ID environment because a Windows Update or an application might have modified or deleted the registry keys.

```powershell
# Unjoins the device from Microsoft Entra ID 
dsregcmd /leave

# Rejoins the device to Microsoft Entra ID
dsregcmd /join

# Automatically unjoins and rejoin the device on Microsoft Entra ID
dsregcmd /forcerecovery
```

It's worth noting that when the device is unjoined and rejoined a login prompt requesting credentials will popup which requires valid user credentials. If you're experiencing issues with leaving or joining the environment it's always worth using `/debug` parameter to troubleshoot why the command is failing.

## Other features

The `dscmdreg` also comes with features to view SSO, Diagnostics Data, and Tenant information which are useful in situations where there is an issue between the device and Microsoft Entra ID.

## Conclusion

The `dsregcmd` is our best-friend when we are troubleshooting any issues related to Microsoft Intune. It allows us to view the device state, single sign on, and tenant information of the device which are extremely useful for troubleshooting issues related to Microsoft Intune.
