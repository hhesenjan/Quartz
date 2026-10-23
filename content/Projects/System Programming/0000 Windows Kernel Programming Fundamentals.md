---
title: "Windows Kernel Programming Fundamentals"
created: 2026-03-01
modified: 2026-03-01
tags: ["C++", "WINDOWS"]
draft: false
---

## Process vs Thread

A process is an object representing a running process and it consists of virtual address space, executable code, and handle table. The threads uses these resources from the process to execute the code. Each thread has its own stack memory within the process memory which is used to handle operations.

## Virtual Memory vs Physical Memory

<image style="display:flex; width: 80%; margin: auto;align-items: center;justify-content: center;" src="0000 Kernel-Programming-Fundamentals-01.svg"/>

Each process has their own virtual memory. A 32-bit process has a virtual memory with an address space of 2GB which can be extended to 3GB using `LARGEADDRESSAWARE`. A 64-bit process has a virtual memory with an address space of 8TB. 

The virtual memory is separated in pages where each page is 4096 bytes. Each pages are also mapped to the physical memory or disk space. Multiple of pages from different processes can be mapped to the same physical memory location as that will save a-lot of memory space.

## System Calls

<image style="display:flex; width: 80%; margin: auto;align-items: center;justify-content: center;" src="0000 Kernel-Programming-Fundamentals-02.svg"/>
 
When a Windows API function like `CreateFile` is called from `kernel32.dll` it actually calls the `NtCreateFile` from `ntdll.dll`. The NTDLL loads the syscall number to the EAX register and then performs a `syscall / sysenter` to switch the thread from user-mode to kernel-mode to perform the `CreateFile` operation requested in user-mode. 

## Handles and Objects

A handle in a process allows the user-mode client to communicate and interact with the kernel. The handle can be viewed as a reference to the kernel object while the kernel object is the one that allows us to perform operations on the system.

<image style="display:flex; width: 70%; margin: auto;align-items: center;justify-content: center;" src="0000 Kernel-Programming-Fundamentals-03.svg"/>

A better way of explaining [[#Handles and Objects]] is through symbolic links, device objects, and driver objects. When a handle is opened using the symbolic link, the kernel returns a handle referencing the device object *(kernel object)*. The device object can then be used to perform operations on the system using the driver object.
