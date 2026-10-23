---
title: "Information Gathering"
created: 2025-04-20
modified: 2025-04-20
tags: ["INFO"]
---

## OSINT

OSINT (Open Source Intelligence) is about gathering information about our target using public resources. It's possible to gather a-lot of information about the company through public resources as many employees are not aware of best security practices.

### Employees

The simplest and easiest way to gather information about a company is by inspecting job posting and social media accounts of employees. Using these resources we can obtain the following information about the company:

* **Vendors, suppliers, and third-parties that deals with the company**
* **Standard laptops, computers, and operating systems the company uses**
* **The technologies the organization uses**
* **The user principal name of employees**

All these informations can be used by threat actors to send spear phishing emails, mimic a vendor, and craft malicious payload for our systems. 

### Domain Name Records

The Domain Name Records consists of a-lot of informations such as owner of domain, subdomains, and TXT records which can be used to obtain the technologies the organization uses.

[CRT.SH](https://crt.sh/) and [Censys](https://search.censys.io/) are that allows us enumerate subdomains and their associated IPv4/IPv6 addresses.

```shell title="CRT.SH"
# Curling information about a website
curl -s 'https://crt.sh/?q=husenjan.com&output=json' | jq . | grep name | cut -d ':' -f 2 | grep -v 'CN=' | cut -d '"' -f 2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u > subdomainlist

# Getting the IP-addresses for these hosts
for i in $(cat subdomainlist);do host $i | grep 'has address' | grep 'husenjan.com' | cut -d" " -f1,4 | sort -u; done
```

```shell title="Censys"
curl https://search.censys.io/search?resource=hosts&sort=RELEVANCE&per_page=25&virtual_hosts=EXCLUDE&q=husenjan.com
```

[Shodan.io](https://www.shodan.io/) is a great tool for enumerating open ports on the IPv4/IPv6 addresses.

```shell title="Obaiting Information with Shodan"
# Capturing only IP-addresses 
for i in $(cat subdomainlist); do host $i | grep 'has address' | grep 'husenjan.com' | cut -d ' ' -f 4 | sort -u; done > iplist

# Using Shodan on these IP-addresses
for i in $(cat iplist); do shodan host $i; done
```

The `dig` command allows us to fetch domain name records such as A records, TXT records, and much more.

```shell title="Obtaining Information with Dig"
dig any 'husenjan.com'
```

The TXT records can contain a-lot informations such as the technologies the organization uses *(Microsoft 365, Google, and etc...)* which are valuable information that can be used to craft phishing emails to bypass these technologies. 

### Cloud Storage

Cloud Storages are widely used in all organization's in some kind of form and these cloud storages can easily be misconfigured by a employee and that will allow the threat actor to access it without any authentication. It's fairly easy to find these cloud storages using Google Search engine.

```shell title="Amazon S3 Bucket & Azure Blobs & Google Cloud Storage"
# Amazon S3 Bucket
intext:'<COMPANY-NAME>' inurl:'amazonaws.com'

# Azure Blobs
intext:'<COMPANY-NAME>' inurl:'blob.core.windows.net'

# Google Cloud Storage
intext:'<COMPANY-NAME>' inurl:'storage.googleapis.com'
```

Inside cloud storage there can be confidential documents, ssh keys, and passwords which are extremely valuable. It's impossible for a organization to track all these activities because the traffic could be from a visitor or employee. If you're interested in learning more about OSINT, I highly recommend checking out the [OSINT Framework](https://osintframework.com/).

## Nmap

Network Mapper (Nmap) is a critical tool for penetration testing as it allows us to obtain information's about our targets. It comes with capabilities such as Host Discovery, Port scanning, and Operating System Detection and so forth... 

It's always recommended to do manual enumeration after obtaining information's from Nmap as there might be security misconfiguration or vulnerabilities which were not detected by Nmap. It's also a good practice to always do manual enumeration as that will allow us to understand the service and the purpose of it which could save us anywhere from hours to days.

**What is Host Discovery?** It's a technique in Nmap that enables us to search through a IP-address range or an list to find active hosts. This is great for situations were we need to search for active hosts in an environment to find potential targets.

```shell title="Host Discovery"
# Nmap: Scans through range of IP-addresses
nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5

# Nmap: Scans through a list of IP-addresses
nmap -sn -oA tnet -iL '<HOST-LIST>' | grep for | cut -d " " -f5

# Nmap: Scans through multiple of IP-addresses
nmap -sn -oA tnet 10.129.2.0-20 | grep for | cut -d " " -f5
```

**What is Port Scanning?** It's a technique in Nmap that enables us to search for open TCP and UDP ports on our targets and obtain information's about the services running on these ports. The main difference between TCP and UDP ports is the TCP port uses three way handshake and ensures that all packet sent from client and server is received while UDP keeps sending packets while delivery of packets fails.

```shell title="Port Scanning"
# Nmap: Scanning for TCP ports
nmap -sV -sC -sT -p- --min-rate=10000 '<RHOST>' --disable-arp-ping --packet-trace

# Nmap: Scanning for TCP ports Half Handshake
nmap -sV -sC -sS -p- --min-rate=10000 '<RHOST>' --disable-arp-ping --packet-trace 

# Nmap: Scanning for UDP ports
nmap -sV -sC -sU -p- --min-rate=10000 '<RHOST>' --disable-arp-ping --packet-trace
```

When Nmap is completed with scanning our targets the ports can be in different states such as open, closed, filtered, and unfiltered because the firewall, intrusion detection system, or intrusion prevention system disallowed our packets from accessing or communicating with these ports. It's possible to bypass these protection by using the following commands.

```shell title="Bypassing Firewalls & IDS & IPS"
# Nmap 1.0: For testing Firewall/IDS/IPS...
nmap '<RHOST>' -n -Pn --disable-arp-ping -p 80

# Nmap 1.1: Bypassing security protection by using decoy IP-addresses
nmap '<RHOST>' -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5 

# Nmap 1.2 : Bypassing security protection by using spoofed IP-address
nmap '<RHOST>' -n -Pn -p 80 -O -S '<SPOOFED-IP>' -e tun0

# Nmap 1.3: Bypassing security protection by using DNS Proxying
nmap '<RHOST>' -n -Pn -sS -p 445  --disable-arp-pint --packet-trace --source-port 53 
```

```shell title="Interacting with Port using DNS Proxy"
ncat -nv --source-port 53 '<RHOST>' '<RPORT>'
```

If our port connection comes back as `Filtered` it means that our packets were blocked by the firewall, intrusion detection system, or intrusion prevention system. We can try the different commands to bypass the security mechanisms and after successfully bypassing it we can use `ncat` to manually enumerate the service.

## Protocols

Understanding the functionality of different protocols such as SSH, FTP, SMB, NFS, DNS, SMTP, IMAP & POP3, SNMP, MySQL, MMSQL, Oracle TNS, and IPMI is important because these are services that will be found in a-lot organization’s and sometimes these services can be misconfigured which allows us to obtain information’s about the organization’s infrastructure.

### SSH

Secure Shell (SSH) is a network protocol that enables us to securely and remotely communicate with our servers and machines through a encrypted tunnel. It’s possible to configure SSH to accept passwords or a RSA key. Here’s a overview of logging into a server using SSH.

```shell title="SSH"
# Logging in using password
ssh <USERNAME>@<IP>

# Logging in using RSA key
ssh -i id_rsa <USERNAME>@<IP>
```

The preferred authentication method in 2025 is RSA key as password is something which can be guessed by a threat actor. However, passwords are still widely used for SSH nowadays.

### FTP

File Transfer Protocol (FTP) is one of the oldest protocols that runs at the application layer. The FTP protocol was first establishes a connection on TCP Port 21 which is used for sending commands and returning error codes. The FTP service then establishes another connection at TCP Port 20 which is used for transferring from user to server and server to user. Here are overview of some commands that can come in good use while working with FTP service.

```shell title="FTP"
# Conencting as anonymous 
ftp anonymous@10.129.15.20

# FTP: Download file
get '<FILE>'

# FTP: Upload file
put '<FILE>'

# FTP: Downloading all files
wget -m --no-passive 'ftp://<USERNAME>@<IP>'
```

In some instances the FTP service can be misconfigured to allow anonymous logins which allows everyone to access the FTP service without authenticating and in these situations the FTP service is a goldmine as threat actors can download confidential files and replace executables with their malicious executable.

### SMB

Server Message Block (SMB) is a client-server protocol that regulates access to files, directories, and other networks such as printer, routers, and interfaces released for the network. An alternative for SMB is Samba which is developed for Unix-based operating systems and it allows Linux systems to communicate with Windows severs. In modern infrastructure the SMB and Samba uses the TCP Port 445 while in legacy infrastructure the UDP Port 137, 138, and 139 are commonly used. Here are some commands which I recommend familiarizing yourself with as SMB service is common in almost all environment’s nowadays.

```shell title="SMB"
# Listing all directories inside SMB client
smbclient -L '\\\\<RHOST>\\'

# SMB: Download a file
get '<FILE>'

# SMB: Upload a file
put '<FILE>'
```

In some environments the SMB service allows the guest user to access shares and this can lead to unauthorized access to confidential files and replace files with their malicious executables.

```shell title="SMB"
# Logging in as guest to SMB service
smbclient -U 'guest' \\\\<IP>\\<SHARE>

# Logging in as guest to SMB service
crackmapexec smb '<IP>' --shares -u 'guest' -p ''
```

After successfully accessing the SMB service it’s possible to obtain information’s about the Windows server using Enum4Linux and RPCClient but that requires us to manually enumerate the SMB service.

```shell title="SMB"
enum4linux '<RHOST>' -A
```

```shell title="RPCClient"
# Logging in as anonymous user with RPCClient
rpcclient -U "" '<RHOST>'

# RPCClient: Getting server information's
srvinfo

# RPCClient: Getting domain information's
enundomains

# RPCClient: Getting more domain information's
querydominfo

# RPCClient: Getting file share information's
netshareenumall

# PRCClient: Getting information about a directory
netsharegetinfo '<DIRECTORY>'

# RPCClient: Getting all users from SMB service
enumdomusers

# RPCClient: Getting user information's from SMB service
queryuser '<HEX-ID>'
```

One thing that should be “must do” is running enum4linux against the SMB service at all penetration tests as it can help us obtain a-lot of information’s about the Windows server. I highly recommend setting up a SMB service and try to understand the service as well as possible as it’s almost used in all environments.

### NFS

Network File System (NFS) is a protocol developed for the same purpose as SMB. NFS is mostly optimized for Linux and Unix operating systems and it comes with different version such as NFS v2, NFSv3, and NFS v4 and each of them comes with different capabilities.

The NFS v2 is supported by many systems and operates over the UDP protocol. The NFS v3 comes with capabilities to handle larger files, better synchronization, and better error reporting. The NFS v4 includes Kerberos, ACL support, performance improvements, and higher security. It’s extremely simple to setup NFS share in our Linux environment.

```shell title="NFS"
# Replace '10.129.10.0' with your IP subnet
echo '/mnt/nfs  10.129.10.0/24(sync,no_subtree_check)' >> /etc/exports

# Restart the NFS service
sudo systemctl restart nfs-kernel-server
```

Once the NFS server is up and running it’s possible to use the following commands to mount and un-mount the NFS share from our client devices.

```shell title="NFS"
# Lists all mounts for NFS
showmount -e '<IP>'

# Mounts the NFS to a directory
sudo mount -t nfs '<IP>:/' '<LOCAL-FOLDER>' -o nolock

# List all files to mounted directory
ls -lna 'LOCATION'

# Unmounts the NFS share
sudo umount '<LOCAL-FOLDER>'
```

When NFS has `no_root_squash` inside `/etc/exports` it will allow threat actors to elevate their privileges using SUID which is uploading the file as root and executing it on the server as a normal user.

### DNS

Domain Name System (DNS) is a protocol for resolving domain names to IP addresses as that allows organizations to setup domains such as marketing.int.husenjan.com and direct the user to internal website. It’s possible for a threat actor to find these IP addresses by enumerating the DNS with Dig and dnsenum.

```shell title="DNS"
# Dig: Nameserver Information
dig ns '<HOSTNAME>' '@<IP>'

# Dig: Version Query
dig ch txt version.bind '<IP>'

# Dig: Query all
dig any '<HOSTNAME>' '@<IP>'

# Dig: Query Zone
dig axfr '<HOSTNAME>' '@<IP>'

# Aggressively scanning for subdomains with dnsenum
dnsenum --dnsserver '<DNS-SERVER>' --enum -p 0 
/opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt '<HOSTNAME>'
```

It’s always recommended to enumerate DNS service as in some instances it’s possible to find out different web application’s and services the organization uses by viewing the TXT records.

### SMTP

Simple Mail Transfer Protocol (SMTP) is a protocol used for sending emails. It’s usually combined with IMAP and POP3 protocols as it allows the user to organize and view their emails. The SMTP service always runs on either TCP Port 25 or TCP Port 587 (Over SSL/TLS). If you’re interested in setting up SMTP service it’s recommended to use ESMTP as it’s more secure and reliable. Here are some commands which are useful to be familiar with to work with SMTP service. 

```shell title="SMTP"
# Telnet: Connecting to SMTP service
telent '<IP>' 25

# SMTP: Communicating
HELO '<HOSTNAME>'
EHLO '<HOSTNAME>'

# SMTP: Finding users
VRFY '<USERNAME>'

# SMTP: Automatically enumerate users
smtp-user-enum -M VRFY -U '<USERNAMELIST>' -t '<TARGET>'

# SMTP: Sending email

MAIL FROM: <mrb3n@inlanefreight.htb>
RCPT TO: <mrb3n@inlanefreight.htb>

DATA

From: <cry0l1t3@inlanefreight.htb>
To: <mrb3n@inlanefreight.htb>
Subject: DB
Date: Tue, 28 Sept 2021 16:32:51 +0200
Hey man, I am trying to access our XY-DB but the creds don't work. 
Did you make any changes there?
.
```

VRFY command may seem completely useless but finding these usernames can be a goldmine as it allows us to launch our brute-force attack against valid usernames. You don’t necessary need to remember these commands but it’s useful to know these incase SMTP service is running on a target.

### IMAP/POP3

Internet Message Access Protocol (IMAP) is a protocol that is assigned the TCP Port 143 and 993 and comes with capabilities for us to access emails, manage emails, and folder structure support. Post Office Protocol Version 3 (POP3) is a protocol that is assigned the TCP Port 110 and 995 and comes with same capabilities as IMAP besides folder structure support. While enumerating IMAP and POP3 services it’s always worth connecting to them TCP and TCP over HTTPS.

```shell title="IMAP/POP3"
# Nmap: Scannign IMAP and POP3
nmap -sV -sC -p 110,143,993,995 '<IP>' -o nmap.result

# Netcat: Connecting to IMAP and POP3 using TCP
ncat -nv '<RHOST>' '<RPORT>'

# OpenSSL: COnencting to IMAPs using TCP over HTTPS
openssl s_client -connect '<IP>:imaps'

# OpenSSL: Connecting to POP3s using TCP over HTTPS
openssl s_client -connect '<IP>:pop3s'
```

Once a connection has been established between our client and the IMAP or POP3 service it’s possible to communicate with the IMAP and POP3 service using the following commands.

**IMAP Commands**

| **Command**                    | **Description**                                        |
| ------------------------------ | ------------------------------------------------------ |
| `1 LOGIN USERNAME PASSWORD`    | Logging into the IMAP service.                         |
| `1 LIST '' *`                  | Listing all mailboxes.                                 |
| `1 CREATE 'INBOX'`             | Creates a mailbox.                                     |
| `1 DELETE 'INBOX'`             | Deleting a mailbox.                                    |
| `1 RENAME 'INBOX' 'Important'` | Renames a mailbox.                                     |
| `1 LSUB '' *`                  | Returns a list of emails the mailbox is subscribed to. |
| `1 SELECT 'INBOX'`             | Selects a mailbox so emails can be accessed.           |
| `1 UNSELECT 'INBOX'`           | Exits the selected mailbox.                            |
| `1 FETCH 1 ALL`                | Retrieves data associated with a email.                |
| `1 FETCH 1 body[text]`         | Retrieving body associated with a email.               |
| `1 LOGOUT`                     | Closes the IMAP server.                                |

**POP3 Commands:**

| **Command**     | **Description**                                             |
| --------------- | ----------------------------------------------------------- |
| `USER username` | Identifies the user.                                        |
| `PASS password` | Authentication of the user using its password.              |
| `STAT`          | Requests the number of saved emails from the server.        |
| `LIST`          | Requests from the server the number and size of all emails. |
| `RETR id`       | Requests the server to deliver the requested email by ID.   |
| `DELE id`       | Requests the server to delete the requested email by ID.    |
| `CAPA`          | Requests the server to display the server capabilities.     |
| `RSET`          | Requests the server to reset the transmitted information.   |
| `QUIT`          | Closes the connection with the POP3 server.                 |

It’s not necessary to remember all these commands for IMAP and POP3 services but I highly recommend noting them down so it’s accessible in the future because it’ extremely useful to have them. If you’re interested in learning more about IMAP commands I recommend checking out [IMAP Commands by DonustherLand](https://donsutherland.org/crib/imap). 

### SNMP

Simple Network Management Protocol is a protocol is used for monitoring and managing network devices such as routers, switches, and IOT devices. The SNMP protocol is assigned UDP Port 161 and it uses the UDP Port 162 t communicate with network devices incase of an event. SNMP also uses Management Information Base (MIB) to support different devices across different manufacturers.

The SNMPv3 is a secure protocol as it comes with authentication and encryption under transmission. However, SNMPv1 and SNMPv2 doesn’t come with these security capabilities and that allows threat actors to access community strings such as public and private. However, the SNMPv3 service can be misconfigured without any authentication which allows us to access strings inside SNMPv3. Here are some commands to enumerate the SNMP protocol.

```shell title="SNMP"
# SNMPWalk: Obtaining information's about SNMPv1
snmpwalk -v 1 -c public '<RHOST>'

# SNMPWalk: Obtaining information's about SNMPv2
snmpwalk -v 2c -c public '<RHOST>'

# SNMPWalk: Obtaining information's about SNMPv2
snmpwalk -v3 -l noAuthNoPriv -c public -m ALL '<RHOST>:<RPORT>'

# Onesixtyone: Brute-force attacking SNMP service for strings
onesixtyone -c /usr/share/wordlists/seclists/Discovery/SNMP/snmp.txt '<RHOST>'
```

In some instances the public and private strings doesn’t exists in SNMP protocol and in these instances it’s possible to use onesixtyone to launch a brute-force attack to find these strings.

### MySQL

MySQL is a database system that enables us to store information’s about our users, website settings, posts, and articles and much more… It’s common for dynamic web applications to fetch and update these data while doing authentication and updating title of the web applications. All these actions are completed using the SQL language. 

```mysql title="MySQL"
# Login to MySQL database
mysql -u '<USERNAME>' -p -h '<RHOST>'

# View all databases
show databases;

# Select a database
use '<DATABASE-NAME>';

# View all tables
show tables;

# View records inside a table
select * from  '<TABLE>';

# View records inside a table with filter
select * from '<TABLE>' WHERE name = 'John Doe';
```

MySQL database contains a-lot of valuable information’s about website, users, and customers and if these information’s are leaked it can significantly damage the reputation of the organization which can have a negative financial effect.

### MSSQL

Microsoft SQL (MMSQL) is a relational database system developed by Microsoft and it’s assigned the TCP Port 1433. MSSQL is popular because it’s compatible with the .NET programming language which is widely used for developing applications. In MSSQL all the actions are completed using the Transact-SQL language.

```mysql
# MSSQL (Local Account): Connecting to MSSQL using Linux
mssqlclient '<USERNAME>@<IP>'

# MSSQL (Domain Account): Connecting to MSSQL using Linux
mssqlclient '<USERNAME>@<IP>' -windows-auth

# List all databases
select name from sys.databases;

# List all tables
select table_name from information_schema.tables;
```

Understanding and learning about MSSQL will be important for us as it’s a database system which is widely used by organization’s. I personally recommend setting up MSSQL on a virtual machine and start playing around with it.

### Oracle TNS

Oracle Transparent Network Substate (Oracle TNS) is a database system from Oracle which is assigned the TCP Port 1521 and it supports protocols such as IPX/PSX and TCP/IP because of that it’s preferred in industries such as healthcare, finance and retails. It also comes with security features such as encryption at rest and transit, comprehensive analysis tools, error reporting, logging, workload management, and fault tolerance.

To access a Oracle TNS it requires us to use a SID and valid credentials otherwise our connection will not be established with database. The SID is used by Oracle’s enterprise application to identify the database. It’s possible to brute-force attack the SID and credentials with the following commands.

```shell title="Oracle TNS"
# Nmap: Brute-force attacking SID
sudo nmap -p 1521 -sV --script oracle-sid-brute '<IP>'

# Odat: Brute-force attacking Credentials
odat.py all -s '<IP>'

# SQLPlus: Logging in to Oracle TNS
sqlplus '<USERNAME>@<IP>/<SID>'

# SQLPlus: Logging into a different databse
sqlplus '<USERNAME>@<IP>/<SID>' as sysdba
```

Accessing the Oracle TNS database requires us to install SQLPlus client from Oracle into our client machine. It’s possible to install the SQLPlus client using the following commands.

```shell title="Oracle TNS"
# Install SQLPlus
sudo apt-get install  oracle-instantclient-sqlplus

# Fix issue with SQLPLus
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";sudo ldconfig
```

SQL language for Oracle TNS is similar to MSSQL database. However, it requires us to use different layout to access the different informations.

```sql title="Oracel TNS"
-- Listing all tables
select table_name from all_tables;

-- See all user privileges
select * from user_role_privs;

-- Extact usernames and passwords
select name, password from sys.user$;
```

Oracle TNS is a popular application for enterprises as it’s compatible with Oracle’s enterprise applications. It’s worth digging deeper into the Oracle TNS if you’re interested learning more about it.

### IPMI

IPMI (Intelligent Platform Management Interface) is a hardware-based management system that is assigned the UDP Port 623 and it’s used for managing and monitoring systems even when they are powered off. It allows IT administrators to monitor temperature, voltage, fan status, and power supplies and it can be setup with SNMP to send alerts and logs. In some instances the IPMI can be accessed through default credentials because IT administrator forgot to change it.

| **Product**     | **Username**  | **Password**                                                |
| --------------- | ------------- | ----------------------------------------------------------- |
| Dell iDRAC      | root          | calvin                                                      |
| HP iLO          | Administrator | Randomized 8 character consisting with numbers and strings. |
| Supermicro IPMI | ADMIN         | ADMIN                                                       |

If the HP iLO is using the default credentials it’s possible to use Hashcat to guess the password with the following command `{bash}hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u`. In some instances it’s possible for us to dump the IPIMI hash using the following commands.

```shell title="IPMI"
# Launch msfconsole
msfconsole

# Use IPMI bruteforce script
use auxiliary/scanner/ipmi/ipmi_dumphashes

# Setting Remote Port
set RHOST '<IP>'

# Executing brute-force attack
run

# Using hashcat to crack password hash
hashcat -m 7300 '<HASH>' /usr/share/wordlists/rockyou.txt
```

IPMI protocol is extremely valuable for threat actors as once they have gotten access to IPMI it’s possible for them to access business critical systems as root and administrator user.

### Remote Management Protocols

In Linux the most common way of accessing a server remotely is SSH and R-Services. SSH is currently the standard protocol in modern systems but in older systems may still be using R-Services. The most common way for accessing Windows server remotely is through Remote Desktop Protocol (RDP), Windows Remote Management (WinRM), and Windows Management Instrumentations (WMI). Here’s a small overview of the different protocols:

| Name                               | Port       | Description                                                                                                                                                                            |
| ---------------------------------- | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Remote Desktop Protocol            | 3389       | A solution that allows administrators to manage system over SSL/TLS.                                                                                                                   |
| Windows Remote Management          | 5985, 5986 | A solution that establishes connection between remote host using the Simple Object Access Protocol (SOAP) over SSL/TLS.                                                                |
| Windows management Instrumentation |            | Allows read and write access to all settings on a Windows systems such as PCs and servers and it’s usually accessed through PowerShell, VBScript, and Windows Instrumentation Console. |

Here is a overview of commands that allows us to access the Windows server remotely using Parrot OS, Kali Linux, and other Linux systems.

```shell title="Remote Management Protocol"
# SSH: Connecting to remote host
ssh '<USERNAME>@<IP>'

# SSH: Connecting to remote host using SSH key
ssh -i id_rsa '<USERNAME>@<IP>'

# Remote Desktop Protocol: Connecting to a remote host
xfreerdp /u:'<USERNAME>' /p:'<PASSWORD>' /v:'<RHOST>'

# Windows Remote Management: Connecting to remote host
evil-winrm -i '<RHOST>' -u '<USERNAME>' -p '<PASSWORD>'

# Windows Management Instrumentation: Connecting to remote host
wmiexec.py '<USERNAME>:<PASSWORD>@<RHOST>' '<HOSTNAME>'
```

It’s a-lot of information’s to grasp but understanding the different way we can access Windows servers is important since in some instances we might have obtained valid credentials where the server is only accessible using evil-winrm.

## Web Reconnaissance

Web Reconnaissance consists of obtaining informations about the web applications to potentially find security misconfiguration and vulnerabilities which can be used to extract confidential informations or to remotely access the server. Web reconnaissance consists of three different stages and these are the following:

* **Identifying asset:** Finding subdomains, directories, IP addresses, and technology company uses for web application.
* **Discovering hidden informations:** Finding backup files, git files, and internal documentations.
* **Analzying the attack surface:** Finding vulnerabilities and configurations which can be used to extract informations or obtain a shell into the server.
* **Gather Intelligence:** Identifying user principal name, email addresses, and patterns that can be exploited.

It’s possible to do these stages actively or passively. If we choose actively it requires us to directly communicate with the server by launching Nmap, Fuzzing, Web Spidering, and so on. And if we choose passively it consists of using third party services such as Google, WHOIS, Web Archive, and Code Repositories. 

### WHOIS

WHOIS is a database that stores informations such as registrar, registrar contact, administrative contact, and nameserver about different domain names.

```shell title="WHOIS"
whois '<RHOST>'
```

With the informations obtained from the WHOIS it’s possible for us to launch a social engineering attack, phishing attack, and understand the services the company uses.

### Subdomains

Subdomains is commonly used by organizations to access internal and external web applications. In some instances these subdomains can be running legacy or vulnerable web applications that can be exploited by threat actors. It’s possible for us to find all organizations subdomains using different tools such as dnsenum/ffuf/assetfinder.

```shell title="Subdomains"
dnsenum --enum '<RHOST>.com' -f /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -r
```

It’s important to use different wordlists to enumerate the subdomains as a single list may not be able to find all the subdomains. DNS Zone Transfer is commonly used for copying all DNS records from primary zone to secondary zone. If the DNS is misconfigured it allows threat actors to use DNS Zone Transfer to obtain all DNS records.

```shell title="Subdomains"
# 1. Getting nameservers
dig ns '<RHOST>'

# 2. Exploiting DNS zone transfer
dig axfr '<RHOST>' '@<NAMESERVER>'
```

DNS Zone Transfer is common in organizations as IT-administrators may not be familiar with best security practices for it. However, for threat actors it allows them to obtain all the subdomains and IP addresses of our web application which is extremely valuable.

### VHOST

Virtual Host (VHOST) is a configuration that comes with Apache, Nginx, and IIS that allows us to host multiple of websites using single host. There are three types of Virtual Hosting:

* **Name-Based Virtual Hosting:** This method relies on HTTP Post Header to distinguish between different websites.
* **IP-Based Virtual Hosting:** This type assigns a unique IP-address to each website hosted on the server. This is preferred option as each websites are isolated from each other.
* **Port-Based Virtual Hosting:** The different websites are assigned ports on the same IP-address. This method requires the user to use a specific port to access the different websites.

It’s possible for us to enumerate these virtual host configurations using tools such as gobuster, feroxbuster, and ffuf. Here’s a overview of the different commands.

```shell title="VHOST"
# Gobuster: Finding VHOST
gobuster vhost -u 'http://<RHOST>:<RPORT>' -w '/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt' --append-domain -t 100

# Ffuf: Finding VHOST
ffuf -u 'http://<RHOST>:<RPORT>' -H 'Host: FUZZ.<RHOST>.com' -w '/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.tx'
```

It’s worth running the different commands as gobuster is excellent for finding virtual hosts but ffuf is also great for finding subdomains which gobuster may have missed. It’s also worth running different wordlists against our targets as other penetration testers more likely already ran the common wordlists against target.

### Fingerprinting

Fingerprinting is about finding out the different technologies a web application uses. This is about finding out the web server, operating system, and software components that the web application as that allows us to search for vulnerabilities for these technologies. Here are some technologies that can be useful to obtain these informations:

| **Tool**    | **Description**                                                                                 |
| ----------- | ----------------------------------------------------------------------------------------------- |
| Wappalyzer  | Identifies a wide range of technologies including CMS, Frameworks, and analytics and much more… |
| BuiltWithin | Provides detailed report about the technology stack.                                            |
| WhatWeb     | Uses vast database of signatures to identify web technologies.                                  |
| Nmap        | Scripts can be used for obtaining fingerprinting informations.                                  |
| Netcraft    | Provides detailed reports on a website’s technology, hosting provider, and security posture.    |
| wafw00f     | Helps determining the WAF is present and the type of configuration.                             |

We can use the following commands on our machines to also obtain these informations. 

```shell title="Fingerprinting"
# Banner Grabbing
curl -I '<RHOST>'

# Detecting WAF
wafw00f '<RHOST>'

# Detecting software components of web application
nikto -h '<RHOST>' -Tuning b
```

### Crawling

Crawling (Spidering) is a automated bot that goes through different sites and collect links and extracts the informations from these links. The recommended algorithms for crawlers are Breadth-First and Depth-First. It’s recommended to use Breadth-First algorithm if you need a broad overview of the web application but if you need specific content and dig deep into the infrastructure the Depth-First algorithm is recommended. The Burp Suite Spider and ReconSpider is excellent for crawling web applications.

```shell title="Crawling"
ReconSpider 'http://<RHOST>'
```

Crawlers can be useful for obtaining API keys, configuration files, backup files, error logs, backup logs. In some instances it might be able to discover hidden urls and find the version of component and the software the website is powered with. It’s important to note that majority of crawlers can be blocked with `robots.txt` file.

### Search Engines

Search Engines is a great way of obtaining email addresses, credentials, confidential documents, and hidden files and folders. We can use the [Google Hacking Database](https://www.exploit-db.com/google-hacking-database) to create specific queries for our needs. Here are some queries that is possible for us to create.

```shell title="Search Engines"
# Finding .pdf files
site:'http://<RHOST>' intext:'confidential document' filetype:'pdf'

# Finding login pages
site:'http://<RHOST>' inurl:'login'

# Finding configuration files
site:'http://<RHOST>' (intext:'.env' OR intext:'.git')
```

This is a great way of obtaining information about a target in a passively way because there is no way of detecting us. It’s also a possibility that Google has indexed confidential data which can be valuable for us.

### WayBackMachine

WayBackMachine is a snapshot database for web application. It allows us to access old version of the web application and in some instances we might be able to obtain confidential documents, credentials, and API keys by inspecting the old version of the application.

