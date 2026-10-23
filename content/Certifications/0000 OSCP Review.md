---
title: "OSCP Review"
created: 2023-10-27
modified: 2023-10-27
tags: ["CERTIFICATION"]
draft: false
---
![[0000 OSCP-Certification-Showcase.png]]
## Introduction

In December 2022, I purchased the Learn One subscription for OSCP. I began studying the content at January 2023 since at the time I was focusing on obtaining other certificates such as:

* [AZ-900](https://learn.microsoft.com/en-us/users/husenjan/credentials/686e367eb51f9614)
* [SC-900](https://learn.microsoft.com/en-us/users/husenjan/credentials/aab886951d2cec68)
* [MS-900](https://learn.microsoft.com/en-us/users/husenjan/credentials/191a48d26936b5f4)

Once I successfully obtained those certificates I shifted my focus solely on OSCP. In the beginning it was difficult to go through the OSCP content since Offensive Security Student Mentors are only allowed to give hints about exercises for the different challenges. This forced me to dig deeper into things which I did not understand and after doing this for a while I got used to *Thinking Out Of The Box* or as Offensive Security says *Try Harder*.

## Restart

When I was about 35% finished with the OSCP course materials, Offensive Security released a new version of [OSCP](https://www.offsec.com/offsec/pen-200-2023/). In the new version the content and the lab machines were significantly improved. The Buffer Overflow section was completely removed on the new PEN-200 after I spent 50-hours or more learning it.

## Try Harder

I started studying the new OSCP (PEN-200) materials at April 2023 when it was released to everyone. I studied for every single day for 4-hours and on Saturdays and Sundays I spent more than 12-hours studying and after doing this for 3 months I had completely completed all the course content. The lab machines took me a month to finish and I managed to own 57/57 lab machines.

## Preperation

Once I completed all the course materials and lab machines, I decided to focus on Proving Grounds and HackTheBox machines to improve my penetration testing skills. Here's a list of Proving Grounds and HackTheBox machines I would highly recommend trying out.

| Platform            | Machines                                                                                                                         |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Proving Grouns      | DRV4, Slort, Heist, EvilBox One, MoneyBox, and Sar                                                                               |
| **HackTheBox**      | MonitorsTwo, Busqueda, PC, Sunday, Irked, Lame, Shocker, Beep, Blue, Networked, Pilgrimage, Sau, Legacy, Optium, Bastard, Sniper |

I completed many more machines but these are some which I recommend doing to prepare for the OSCP exam. After successfully compromising these machines, I decided to book my exam and from there I solely focused on improving my notes for the exam.

## Exam

At the time of the exam, I was about 15-minutes late since I thought the md5 hash would be sent to me on email 30-minutes before the exam starts. However, that wasn't the case since the md5 hash was included included in the booking confirmation email.

In the beginning of the exam I was struggling for two hours to obtain the initial foothold on the active directory machines because I was following an never ending rabbit hole. After taking a break for 10 to 15 minutes I came back with a clear mind and managed to obtain the initial foothold for the active directory machine from there I successfully managed to lateral move through the active directory environment and compromise the domain controller.

Once the active directory set was compromised I decided to focus on compromising the two Linux machines and these machines were fairly easy to compromise as I was extremely familiar with Linux systems and the most common methodologies to compromise web applications.

Once I compromised these two Linux machines I decided to focus on the Windows machine which is not domain-joined and the initial foothold for the machine was fairly simple but the privilege escalation took me a really long time and after spending hours and hours, I successfully managed to compromise it. Here's a quick overview of the time I spent to compromise all the different machines:

* **Active Directory Set:** 5 Hours
* **Linux Machines Standalone (2 Machines):** 4 Hours
* **Windows Machine Standalone(1 Machine):** 5 Hours

After successfully compromising all the machines I spent some time going through my notes and taking screenshots to make sure that I didn't miss anything for the report because Offensive Security (Offsec) requires that you use the following commands once you compromise a machine `whoami`, `hostname`, `cat <FLAG-LOCATION>` otherwise you may not receive the points for compromising the machine. 

I spent around 16 hours to complete the report and after delivering it in it took Offensive Security (Offsec) around two days to review it and confirm that I successfully passed the [OSCP certification](https://www.credential.net/b96accfa-fc61-44bd-b439-ce05d2c3a7b9#acc.g7JP5UNj). I became extremely happy after passing OSCP since I spent the last 7 months studying for the certification. However, I know this is just the beginning of my long journey!