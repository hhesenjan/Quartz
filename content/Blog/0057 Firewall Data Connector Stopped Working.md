---
title: "Firewall Data Connector Stopped Forwarding Logs to Microsoft Sentinel"
created: 2026-09-26
modified: 2026-09-26
tags: ["SIEM", "SOC", "SENTINELONE"]
draft: false
---

## Introduction

Well recently we had a issue with the data connector which was forwarding our firewall logs to Microsoft Sentinel. The issue was related to the disk space being full at the point nothing could run on the system. Unfortunately, there were no documentation for the way it was configured or the right way of increasing the disk space since in Linux there are multiple of ways to increase disk space. In this article I'll go through increasing the disk space. 

*Please keep in mind that it has been some years since I worked with Linux servers...*

## Increase Disk Space

Our Linux VM which is responsible for forwarding firewall logs uses EXT4 partition so I'll go through increasing disk space for EXT4 extension.

| Step | Description                  | Command                               |
| ---- | ---------------------------- | ------------------------------------- |
| 1    | View disk space              | `lsblk`                               |
| 2    | View filesystem type         | `df -Th /dev/sda1`                    |
| 3    | Grow partition               | `sudo growpart /dev/sda <PARTITION>`  |
| 4    | Verify partition expansion   | `lsblk /dev/sda`                      |
| 5    | Apply the growth on parition | `sudo resize2fs /dev/sda1`            |
| 6    | Verify expansion             | `df -h /dev/sda1`                     |
| 7    | Reboot rsyslog               | `systemctl restart rsyslog`           |
| 8    | Reboot Azure Monitor Agent   | `systemctl restart azuremonitoragent` |

Once the disk space and the different services were rebooted the data connector started working again.

## Decreasing Disk Space

This section of the document goes through decreasing disk space. Its crucial to note that this step requires us to deattach the disk and then resize it with a different VM.

| Step | Description                    | Command                                                                                                                   |
| ---- | ------------------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| 1    | View disk space                | `lsblk -f`                                                                                                                |
| 2    | Check filesystem               | `e2fsck -f /dev/sda2`                                                                                                     |
| 3    | View minimum size required     | `resize2fs -P /dev/sda2`                                                                                                  |
| 4    | Shrink the size                | `resize2fs /dev/sda2 60G`                                                                                                 |
| 5    | Verify the shrink of partition | `e2fsck -f /dev/sda2`                                                                                                     |
| 6    | View block and block size      | `tune2fs -l /dev/sda2 \| grep -E 'Block (count\|size)'`<br>                                                               |
| 7    | Shrink the partition           | `parted /dev/sda`<br>`unit s`<br>`print`<br>`resizepart 2 <End>s`                                                         |

I also decided to include instructions to decrease size since it might help with reducing cost once you have figured out why the Linux VM consumes a-lot  of disk space.

## Conclusion

I primarily focus on things related to cybersecurity but this situation was unique since none knew the reason the data connector stopped working and there were little to none documentation. Therefore I decided to dive into the issue and resolved it by incresing the disk space. I think this information might be helpful for others who might be experiencing similar issue.
