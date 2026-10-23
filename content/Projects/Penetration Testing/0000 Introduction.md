---
title: "What is Pentesting?"
created: 2025-04-15
modified: 2025-04-15
permalink: "Introduction"
tags: ["INFO"]
author: Husenjan
---

## What is Penetration Testing?

Penetration Testing (Pentest) is a organized and authorized attack which the purpose is about finding security vulnerabilities in an organization and use these security vulnerabilities to gain access to the organization’s systems and then further elevate our privileges by compromising more systems.

The main goal of penetration testing is to find these vulnerabilities and have them patched before it affects the organization in a negative way. It’s also possible the organization may choose to accept the risk of a system vulnerability in [[0000 Introduction#Risk Management|Risk Management]].

## Penetration Testing Framework

![[9999 Penetration Testing Framework.svg]]

Penetration Testing Framework is a fantastic framework which goes through the different processes that are included in a penetration test. Here's a quick summary of the different stages:

| **Stages**                           | **Description**                                                                                                                                                |
| ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Information Gathering**            | This stage is about searching for information such as the systems, operating systems, and software’s the organization uses.                                    |
| **Vulnerability Assessment**         | This stage is about researching the vulnerabilities these systems, operating systems, and software’s can be vulnerable to.                                     |
| **Exploitation & Post Exploitation** | This stage is about reading through the exploit code and modifying it to our needs to exploit the service running on the system.                               |
| **Lateral Movement**                 | This stage is about elevating our access by compromising higher privileged accounts which allows us to access systems that contains confidential informations. |

Penetration Testers mostly spends their time in the information gathering and vulnerability assessment stages as these are key stages that allows us to compromise a environment and systems in that environment. And sites such as [CVEDetails](https://www.cvedetails.com/), [Packetstorm](https://packetstorm.news/), [ExploitDB](https://www.exploit-db.com/), [Vulners](https://vulners.com/), and [NIST](https://nvd.nist.gov/vuln/search?execution=e2s1) are great in these stages.

## Risk Management

Risk Management is about analyzing the different security issues and risks to ensure the organization is protected. In some instances the organization may choose to accept certain risks as the system or software is business critical and in those instances it's important to have insurance on those systems. 

Risk Management also plays a critical role in a penetration testing since the system could be vulnerable for multiple of vulnerabilities and executing these vulnerabilities against the system could potentially crash the system and affect the business negatively. An example is if the system is vulnerable for remote code execution and SQL injection, it’s always recommended to use SQL injection to minimize the risks of crashing the system.

## Open Web Application Security Project (OWASP)

Open Web Application Security Project (OWASP) has a list over the top 10 most dangerous vulnerabilities for web applications. It’s a fantastic resource for understanding the different vulnerabilities for web applications and it also includes prevention methods for us to protect our web applications.

| **Number** | **Category**                                                                                                               | **Description**                                                                                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1          | [Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)                                           | Restriction from viewing sensitive information’s about users email, data, and etc…                                                                                            |
| 2          | [Cryptographic Failure](https://owasp.org/Top10/A02_2021-Cryptographic_Failures/)                                          | Failing to encrypt data at rest and transfer can put the data in dangerous people’s hands.                                                                                    |
| 3          | [Injection](https://owasp.org/Top10/A03_2021-Injection/)                                                                   | Validating users input is important as users could try to exfiltrate data from the web application or services.                                                               |
| 4          | [Insecure Design](https://owasp.org/Top10/A04_2021-Insecure_Design/)                                                       | These issues occurs when application is not designed with security in mind.                                                                                                   |
| 5          | [Security Misconfiguration](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/)                                   | Misconfiguring services, cloud services, error logs, and configuration files cloud lead to disclosing too much information’s.                                                 |
| 6          | [Vulnerable and Outdated Components](https://owasp.org/Top10/A06_2021-Vulnerable_and_Outdated_Components/)                 | Using components which are outdated both client-side and server-side.                                                                                                         |
| 7          | [Identification and Authentication Failures](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/) | Implementing protection such as lockout, cooldown, and secure coding practices in authentication aspect of our application can prevent attackers from compromising our users. |
| 8          | [Software and Data Integrity Failures](https://owasp.org/Top10/A08_2021-Software_and_Data_Integrity_Failures/)             | When an web application relies on plugins, libraries, and modules it’s important to ensure it comes from a trusted sources, repositories, content delivery networks.          |
| 9          | [Security Logging and Monitoring Failures](https://owasp.org/Top10/A09_2021-Security_Logging_and_Monitoring_Failures/)     | Security logs and monitoring failures can help us detect malicious activities and prevent them from escalating further.                                                       |
| 10         | [Server-Side Forgery](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)                            | Server-Side Forgery (SSRF) allows a threat actor to fetch internal websites and sources if the user input isn’t validated.                                                    |

It’s important to note that the [OWASP Top 10](https://owasp.org/Top10/) could be different in the near future as new technologies comes out. I highly recommend becoming familiar with the [OWASP Top 10](https://owasp.org/Top10/) if you're interested in specializing in bug bounty hunting.