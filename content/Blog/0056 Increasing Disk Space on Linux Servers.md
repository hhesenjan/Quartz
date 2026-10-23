---
title: "Increasing Disk Space on Linux Servers"
created: 2026-09-26
modified: 2026-09-26
tags: ["SIEM", "SOC"]
draft: false
---

## Introduction

I'll be completely honest I really like Linux as a operating system. However, the one thing I really dislike about it is increasing disk space through CLI since there are so many ways it can be done and there are multiple of extensions to choose from. To make sure that I don't forget about increasing disk spaces in Linux, I decided to create this article.

*Please keep in mind that it has been some years since I worked with Linux servers...*

## Increase Disk Space

You can use the following commands to increase the disk space for the EXT4 extension.

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

If you have any services running on the server, I would recommend rebooting them and they should start working without any issues. 

## Decreasing Disk Space

If you think increasing disk space is a-lot of work well... I got terrible news since in Linux to decrease the disk space it needs to be resized through a different VM.

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

I primarily use Windows as a main operating system since it works out of the box. However, after writing this article, I realized that my Linux skills have been degrading and I'll be focusing on improving the Linux skills going forward by using it as a primary operating system. Hopefully, this article is useful to someone who needs it.
