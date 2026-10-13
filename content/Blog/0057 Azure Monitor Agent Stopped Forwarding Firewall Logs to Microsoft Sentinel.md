---
title: "Increasing Disk Space on Linux Systems"
created: 2026-09-26
modified: 2026-09-26
tags: ["SIEM", "SOC"]
draft: false
---

## Introduction

I think one of the most frustrating thing about Linux systems is when they run out of the disk space and the only way to increase it is through the CLI. In this document I'll og through increasing the disk space for Linux systems.

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

I use Windows as a main operating system since its a system that just works out of the box. However, after writing this article and seeing the knowledge I lost about Linux it makes me want to use Linux as a primary operating system going forward. I think having Linux skill is crucial going forward since majority of critical infrastructure runs on it. 

