---
title: "Exploiting Active Directory Certificate Service"
created: 2024-12-29
modified: 2024-12-29
tags: ["WIN", "AD"]
draft: false
---

## Information

Active Directory Certification Services (ADCS) is a service which is widely deployed in all organization's. It's a service which is not looked into much by security researchers. I recently read about the vulnerabilities the service has and I decided to dive deeper into the service. 

## Understanding The Basics

The Active Directory has a attribute `msDS-KeyCredentialLink` that stores raw cryptographic data for password-less authentication for a user or a computer. It also requires the account to have `pre-authentication` enabled on their account so it can do authentication with certificates. We can use `msDS-KeyCredentialLink` attribute to obtain shadow credentials and that will allow us to maintain access to the account even when the user changes their credentials.

## Prerequisites

An organization with the following requirement's could be vulnerable for multiple of Active Directory Certi-ficate exploits if the environment isn't properly managed and misconfigured:

* Domain Controller with Windows Server 2016 or later
* Domain Controller with Active Directory Services configured with PKINIT support

## Obtaining Shadow Credentials

When the environment has all our prerequisities than that enables us to create shadow credentials by writing to the `msDS-KeyCredentialLink` attribute. This means that we need an account which have `GenericAll` or `GenericWrite` over the account which we want to obtain shadow credentials from. When an account has these privileges it can create a certificate and write the public key to the `msDS-KeyCredentialLink` attribute.

```bash
ceritpy-ad shadow add -u 'fredrik' -p 'password' -t 'int.cosmo.com' -account 'olivia' 
```

What that command does is creating a certificate and writing the public key to the `msDS-KeyCredentialLink` attribute and downloading the private key to our Kali Linux system. We can use the private certificate to obtain the ntlm hash of Olivia using the following command: 

```bash
ceritpy-ad auth -u 'olivia' -pfx management_svc.pfx -d 'int.cosmo.com'
```

The ntlm hash will enable us to authenticate as Olivia and login to all servers which the user has access to by using the pass-the-hash method. Here's an example of evil-winrm being used to authenticate to the domain controller.

```bash
evil-winrm -i 'int.cosmo.com' -u 'olivia' -H 'NTLM HASH'
```

Obtaining shadow credential might seem as a complicated task but misconfiguration happens in an Active Directory environment such as a account accidentally being added into a ACL which has `GenericWrite` over a service account. The service account might have enough privileges to compromise the whole environment.

## Different Vulnerabilities with ADCS

It's important to configure the Active Directory Certificate Service using best practices as misconfiguration can lead to the Active Directory Certificate Service being vulnerable for  ESC1, ESC2, ESC3, ESC4, ESC5, ESC6, ESC7, ESC8 ESC9, ESC10, ESC11, and ESC12. You can read about these vulnerabilities from [Certified Pre-Owned - Will Schroeder](https://posts.specterops.io/certified-pre-owned-d95910965cd2).

I'll go through ESC8 *(Vulnerable Certificate Authority Access Control)* as an example to showcase the vulnerabilities with ADCS. The ESC8 is an great example as it allows us to laterally move through the Active Directory environment by exploiting misconfigured web enrollment template. Here's an example of our environment:

**Information**
* Fredrik has `GenericWrite` over Olivia
* Olivia can manage the ClientAuthentication template
* ClientAuthentication template has domain computer enrollment and client authentication enabled

In an environment such as these it's possible to obtain the ntlm hash of any user. We first need to change the `userPrincipal` of Olivia to administrator, and then request a certificate as olivia, and then change the `userPrincipal` back to it's original value. We can then use the private key of administrator to obtain the ntlm hash.

```bash title="Example of ESC8"
# 1. Changing the userPrincipal of olivia to administrator
certipy-ad account update -u 'fredrik' -p 'password' -t 'int.cosmos.com' -user 'olivia' -upn 'administrator'

# 2. Requesting certificate as olivia
certipy-ad req -ca 'cosmos-DC01-CA' -template 'CertifiedAuthentication' -u 'olivia@int.cosmos.com' -hashes 'NTLM HASH'

# 3. Changing the userPrincipal of olivia to it's original value
certipy-ad account update -u 'fredrik' -p 'password' -t 'int.cosmos.com' -user 'olivia' -upn 'olivia'

# 4. Authenticating as administrator to obtain ntlm hash
certipy-ad req -u 'administrator' -pfx 'administrator.pfx' -d 'int.cosmos.com'
```

All these commands will help us with obtaining the ntlm hash of the administrator. How can we protect our environment from such attack? We can prevent it by enabling HTTPS on the server and disabling ntlm authentication using Group Policy Object.

## References

* NightFox, 2024, [Exploiting and Detecting Shadow Credentials and msDS-KeyCredentialLink in Active Directory](https://medium.com/@NightFox007/exploiting-and-detecting-shadow-credentials-and-msds-keycredentiallink-in-active-directory-9268a587d204), (Accessed on 31 December 2024)
* TheHacker, 2024, [Shadow Credentials](https://www.thehacker.recipes/ad/movement/kerberos/shadow-credentials), (Accessed on 31 December 2024)
* Will Schroeder, 2021, [Certified Pre-Owned](https://medium.com/@NightFox007/exploiting-and-detecting-shadow-credentials-and-msds-keycredentiallink-in-active-directory-9268a587d204), (Accessed on 31 December 2024)